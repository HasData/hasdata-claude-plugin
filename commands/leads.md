---
description: Build a local lead list of businesses for a category in a city — name, phone, address, website, rating
argument-hint: <category> in <city>  [n=N]   (e.g., "plumbers in Brooklyn, NY n=50")
allowed-tools:
  - Bash(hasdata *)
  - Bash(jq *)
  - Bash(wc *)
  - Bash(mkdir *)
---

# /hasdata:leads

Local lead query: **$ARGUMENTS**

## Steps

1. **Parse** the input:
   - `<category>` — the business type
   - `<city>` — the location (after the word `in`)
   - `n=<N>` (optional, default `30`) — target number of leads after dedup

   If `n=` is omitted **and** the user hasn't given a count anywhere in the prompt, ask: *"How many leads do you want? (default 30)"* and wait for an answer before fetching.

   If the format `<category> in <city>` isn't followed, ask the user to clarify before proceeding.

   Compute pagination from the target. Each Maps page yields ~20, each YellowPages page yields ~30. Add headroom for dedup, then cap:

   ```bash
   N=30                           # from n=<N>, or user reply, or default
   MAPS_PAGES=$(( (N + 19) / 20 + 1 )); [ $MAPS_PAGES -gt 5 ] && MAPS_PAGES=5
   YP_PAGES=$(( (N + 29) / 30 + 1 ));   [ $YP_PAGES   -gt 5 ] && YP_PAGES=5
   ```

2. **Find center coordinates for the city.** If you don't already know lat/lng for the city, do one quick lookup:

   ```bash
   mkdir -p .hasdata
   hasdata google-maps --q "<city center>" --pretty -o .hasdata/leads-geocode.json
   ```

   Pull a representative `@lat,lng,12z` from the first result's `gpsCoordinates` and reuse for the actual search.

3. **Paginate Google Maps** — `--start` accepts multiples of 20 (0, 20, 40, …). Fetch `MAPS_PAGES` pages:

   ```bash
   for i in $(seq 0 $((MAPS_PAGES - 1))); do
     start=$(( i * 20 ))
     hasdata google-maps --q "<category>" --ll "@<lat>,<lng>,12z" --start $start \
       --pretty -o ".hasdata/leads-maps-$start.json" &
   done
   wait
   ```

   Stop fetching when a page returns no `places` — Maps tops out around 60–120 results per query.

4. **Paginate YellowPages** for service-trade categories (plumbers, electricians, dentists, contractors, lawyers, etc.). Skip for non-trade categories like restaurants:

   ```bash
   for page in $(seq 1 $YP_PAGES); do
     hasdata yellowpages-search --keyword "<category>" --location "<city>" \
       --sort averageRating --page $page \
       --pretty -o ".hasdata/leads-yp-$page.json" &
   done
   wait
   ```

5. **Combine, normalize, and dedupe.** Use `jq` to flatten across pages and dedupe by `(title, phone)`. Define a reusable function so it can be re-run after the expansion pass:

   ```bash
   merge_leads() {
     {
       for f in .hasdata/leads-maps-*.json .hasdata/leads-maps-similar-*.json; do
         [ -f "$f" ] || continue
         jq -r '.places[]? | [.title, .phone, .address, .website, (.rating // ""), (.reviewsCount // ""), "google-maps"] | @tsv' "$f"
       done
       for f in .hasdata/leads-yp-*.json .hasdata/leads-yp-similar-*.json; do
         [ -f "$f" ] || continue
         jq -r '.places[]? | [.title, .phone, .address, .website, (.rating // ""), (.reviewsCount // ""), "yellowpages"] | @tsv' "$f"
       done
     } | awk -F'\t' '!seen[$1"|"$2]++' \
       | sort -t$'\t' -k5,5nr -k6,6nr \
       > .hasdata/leads.tsv
   }

   merge_leads
   wc -l .hasdata/leads.tsv
   ```

5a. **If short of target — auto-expand to similar categories.** When the deduped count is below `N × 0.8`, brainstorm 3–5 adjacent categories that target the same buyer or solve the same problem. Examples:

   - `plumbers` → `plumbing services`, `drain cleaning`, `water heater repair`, `pipe repair`, `sewer service`
   - `dentists` → `orthodontists`, `dental clinics`, `periodontists`, `family dentistry`
   - `coffee shops` → `cafes`, `espresso bars`, `roasters`, `breakfast places`
   - `yoga studios` → `pilates studios`, `barre studios`, `meditation centers`, `wellness centers`
   - `lawyers` → `law firms`, `legal services`, `attorneys`, `legal consultants`
   - `restaurants` → cuisine-specific subtypes (`italian restaurants`, `sushi`, `mediterranean`, …) only if the user gave a generic term

   Do **not** invent loosely-related categories — they need to be plausibly the same lead pool. If the original category is already specific (e.g. `endodontists in Boston`), skip the expansion and let the count be what it is.

   Run one page per source per similar category, in parallel:

   ```bash
   declare -a SIMILAR=( "plumbing services" "drain cleaning" "water heater repair" )
   for cat in "${SIMILAR[@]}"; do
     safe=$(echo "$cat" | tr ' ' '-')
     hasdata google-maps --q "$cat" --ll "@<lat>,<lng>,12z" --start 0 \
       --pretty -o ".hasdata/leads-maps-similar-$safe.json" &
     hasdata yellowpages-search --keyword "$cat" --location "<city>" \
       --pretty -o ".hasdata/leads-yp-similar-$safe.json" &
   done
   wait

   merge_leads
   wc -l .hasdata/leads.tsv
   ```

   **Cap at one expansion round.** Do not recurse — if it's still short after the similar-categories pass, accept the count and continue.

6. **Present a markdown table** with the top **5** leads sorted by rating × reviews. Columns: Name | Phone | Address | Website | Rating | Reviews | Source. Note the total dedup count above the table (e.g. "Top 5 of 142 leads · full list at `.hasdata/leads.tsv`"). Do not preview more than 5 — the rest stays in the TSV.

   If the expansion pass ran, briefly note which similar categories were folded in (one line, e.g. *"Expanded with: plumbing services, drain cleaning, water heater repair"*).

7. **Offer follow-ups (skip the ones not applicable):**
   - Fan out to full Google Maps profiles for the top N (`google-maps-place` — 5 credits each, returns hours, full reviews summary, photos)
   - Pull recent reviews for any specific business (`google-maps-reviews`)
   - Enrich any lead's owner / contact via `/hasdata:enrich`

   Do not suggest widening the radius, fanning out per-borough/region, or rephrasing the category — the expansion pass in step 5a already handled coverage.

## Cost

5 credits per page per source. Cost scales with `n`:

- `n=20` → 2 Maps + 2 YP pages → ~20 credits
- `n=30` (default) → 3 Maps + 2 YP pages → ~25 credits
- `n=60` → 4 Maps + 3 YP pages → ~35 credits
- `n=100` → 5 Maps (cap) + 4 YP pages → ~45 credits
- `n=200` → 5 Maps (cap) + 5 YP pages (cap) → ~50 credits

Auto-expansion (step 5a) adds ~10 credits per similar category × 2 sources = ~20–50 extra credits when it fires (3–5 categories). Caps at one round.
