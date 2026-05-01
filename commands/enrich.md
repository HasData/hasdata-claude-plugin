---
description: Enrich a person or company with Google search dorks — emails, LinkedIn, GitHub, X (with follower counts), Crunchbase
argument-hint: <name and/or company>  (e.g., "Roman Milyushkevich hasdata" or "Acme Corp")
---

# /hasdata:enrich

Subject: **$ARGUMENTS**

Find the canonical identity (knowledge graph, official site), then enrich with targeted `site:` dorks. **Goal: precision over recall.** It's better to return three confirmed signals than a dozen unverified ones.

## Steps

### 1. Parse input

Detect whether $ARGUMENTS is `Name + Company`, `Name only`, or `Company only`. Set:

```bash
mkdir -p .hasdata/enrich
slug=$(echo "$ARGUMENTS" | tr '[:upper:]' '[:lower:]' | sed -E 's|[^a-z0-9]+|-|g; s|^-||; s|-$||' | cut -c1-60)
NAME="<quoted full name or empty>"      # e.g. '"Roman Milyushkevich"'
COMPANY="<company token or empty>"      # e.g. 'hasdata'
```

### 1a. Disambiguate before enriching (MANDATORY for name-only input)

A name without a company is almost never unique. Names like "Nikita Naumov", "John Smith", "Maria Garcia" map to many real people, and dork results conflate their facts. **Do not enrich a name-only query directly.** Instead:

If `NAME` is set and `COMPANY` is empty, run a **discovery pass** first:

```bash
hasdata google-serp --q "${NAME}" --num 10 --pretty -o ".hasdata/enrich/$slug-disco-1.json"
hasdata google-serp --q "site:linkedin.com/in ${NAME}" --num 10 --pretty -o ".hasdata/enrich/$slug-disco-2.json" &
hasdata google-serp --q "site:wikipedia.org ${NAME}" --num 5 --pretty -o ".hasdata/enrich/$slug-disco-3.json" &
wait
```

Cluster the results into **distinct candidate identities** by inspecting each LinkedIn `/in/<slug>` profile, Wikipedia entry, and personal-domain hit. For each candidate extract:

- A short role / affiliation hint (from the snippet or LinkedIn headline)
- The primary domain or institution
- A 1-line distinguisher

Then **stop** and present a numbered list. Use this exact template:

```markdown
"<NAME>" matches multiple people. Pick one or re-invoke with a disambiguator (employer, role, city):

1. {role / affiliation} — {domain or institution} — {distinguisher}
2. {role / affiliation} — {domain or institution} — {distinguisher}
3. {role / affiliation} — {domain or institution} — {distinguisher}

Reply with the number, or run: /hasdata:enrich {NAME} {employer-or-disambiguator}
```

After printing the list, **do not run any further dorks**. Wait for the user's reply.

When the user picks (`1`, `2`, …) or re-invokes with a disambiguator, set `COMPANY` (or use the institution hint as the anchor) and continue from step 2.

If only **one** clear candidate emerges from discovery (single dominant domain / Wikipedia entry / LinkedIn match), skip the picker and continue automatically — but state in one line which entity was selected.

### 2. Identity pass — find the canonical entity

Run **one** broad query with full `google-serp` (10 results) and inspect the knowledge graph + top organic. This anchors everything that follows.

```bash
hasdata google-serp --q "${NAME} ${COMPANY}" --num 10 \
  --pretty -o ".hasdata/enrich/$slug-identity.json"
```

Pull the knowledge graph (when Google has structured data, this gives website, description, social profiles, founders for free):

```bash
jq '.knowledgeGraph // empty' ".hasdata/enrich/$slug-identity.json"
jq -r '.knowledgeGraph.socialProfiles[]?.link // empty' ".hasdata/enrich/$slug-identity.json"
```

**Determine the company domain** (e.g. `hasdata.com`). Prefer in this order:

1. `knowledgeGraph.website` if present
2. The first organic result whose hostname starts with the company token (`hasdata.com`, not `hasdata.io.example.com`)
3. Ask the user if neither is found

Set `DOMAIN=<company.com>` and use it in subsequent dorks.

### 3. Targeted dorks

Each dork has **one** job. Run all in parallel with full `google-serp` (10 results — the extra context surfaces Twitter cards with follower counts and Knowledge-Graph chips that `google-serp-light` strips):

```bash
declare -a DORKS=()

if [ -n "$NAME" ] && [ -n "$COMPANY" ]; then
  DORKS+=(
    "linkedin-personal::site:linkedin.com/in ${NAME} ${COMPANY}"
    "linkedin-company::site:linkedin.com/company ${COMPANY}"
    "github-user::site:github.com ${NAME}"
    "github-org::site:github.com ${COMPANY}"
    "x-handle::(site:x.com OR site:twitter.com) ${NAME} ${COMPANY}"
    "x-company::${COMPANY} (twitter OR x.com) followers"
    "email-domain::${NAME} \"@${DOMAIN}\""
    "site-mention::site:${DOMAIN} ${NAME}"
    "crunchbase::site:crunchbase.com ${COMPANY}"
  )
elif [ -n "$NAME" ]; then
  DORKS+=(
    "linkedin-personal::site:linkedin.com/in ${NAME}"
    "github-user::site:github.com ${NAME}"
    "x-handle::(site:x.com OR site:twitter.com) ${NAME}"
  )
else
  DORKS+=(
    "linkedin-company::site:linkedin.com/company ${COMPANY}"
    "github-org::site:github.com ${COMPANY}"
    "x-company::${COMPANY} (twitter OR x.com) followers"
    "email-domain::\"@${DOMAIN}\" contact"
    "site-mention::site:${DOMAIN} contact"
    "crunchbase::site:crunchbase.com ${COMPANY}"
  )
fi

for entry in "${DORKS[@]}"; do
  tag="${entry%%::*}"
  q="${entry#*::}"
  hasdata google-serp --q "$q" --num 10 \
    --pretty -o ".hasdata/enrich/$slug-$tag.json" &
done
wait
```

### 4. Extract per-channel — and validate slugs

Each channel uses **strict** filters; only signals that pass the validator survive.

#### LinkedIn (personal)

Only accept `linkedin.com/in/<slug>` where the slug contains a recognizable name token (or appears in a result whose snippet mentions the company). Strip country prefixes (`vg.`, `uk.`, etc.) — they alias the canonical domain.

```bash
jq -r '(.organicResults // [])[] | "\(.link)\t\(.title)\t\(.snippet // "")"' \
  ".hasdata/enrich/$slug-linkedin-personal.json" 2>/dev/null \
  | grep -Ei 'linkedin\.com/in/' \
  | sed -E 's|https?://[a-z]{2,3}\.linkedin\.com|https://linkedin.com|; s|\?.*||' \
  | awk -F'\t' -v name="${NAME//\"/}" -v co="${COMPANY}" 'BEGIN{IGNORECASE=1}
      tolower($0) ~ tolower(co) || tolower($1) ~ tolower(name) {print}'
```

Keep at most the top match. Note: a single `linkedin.com/in/<slug>` URL where `<slug>` doesn't contain *any* name token AND the snippet doesn't mention the company is **noise** — drop it.

#### LinkedIn (company)

```bash
jq -r '(.organicResults // [])[] | .link' ".hasdata/enrich/$slug-linkedin-company.json" 2>/dev/null \
  | grep -Ei 'linkedin\.com/company/' \
  | sed -E 's|https?://[a-z]{2,3}\.linkedin\.com|https://linkedin.com|; s|\?.*||; s|/$||' \
  | sort -u | head -3
```

If multiple slugs surface, prefer the one matching `<company>` exactly, then the longest match.

#### GitHub

```bash
# User profiles (one path segment)
jq -r '(.organicResults // [])[] | .link' ".hasdata/enrich/$slug-github-user.json" 2>/dev/null \
  | grep -Eo 'https?://github\.com/[A-Za-z0-9][A-Za-z0-9-]{0,38}/?$' \
  | sed -E 's|/$||' | sort -u

# Orgs (also one path segment, surfaced by github-org dork)
jq -r '(.organicResults // [])[] | .link' ".hasdata/enrich/$slug-github-org.json" 2>/dev/null \
  | grep -Eo 'https?://github\.com/[A-Za-z0-9][A-Za-z0-9-]{0,38}/?$' \
  | sed -E 's|/$||' | sort -u
```

Only keep handles whose slug matches a token from name or company.

#### X / Twitter — pick the canonical handle by follower count

`google-serp` surfaces inline Twitter cards (with follower count) under `inlineTwitter` or in the knowledge graph for branded entities. Pull all candidate handles, then rank by followers:

```bash
# Candidate handles from organic + inline twitter blocks
{
  jq -r '(.organicResults // [])[] | .link' .hasdata/enrich/$slug-x-*.json 2>/dev/null
  jq -r '(.inlineTwitter // [])[]?.profile?.link // empty' .hasdata/enrich/$slug-x-*.json 2>/dev/null
  jq -r '(.knowledgeGraph.socialProfiles // [])[]? | select(.name | test("twitter|x"; "i")) | .link' \
    .hasdata/enrich/$slug-identity.json 2>/dev/null
} | grep -Eo 'https?://(twitter|x)\.com/[A-Za-z0-9_]{1,15}' \
  | grep -viE '/(home|search|i|share|intent|notifications|messages|explore)$' \
  | sort -u

# Follower counts for any inline twitter cards
jq -r '(.inlineTwitter // [])[]? | "\(.profile.link // "")\t\(.profile.followers // "")"' \
  .hasdata/enrich/$slug-x-*.json 2>/dev/null \
  | grep -v '^\t' | sort -u
```

If two candidate handles surface (e.g. `@hasdata` and `@hasdata_com`), prefer:
1. The handle in `knowledgeGraph.socialProfiles` (Google's verified link)
2. The handle with the higher follower count from `inlineTwitter`
3. The handle that matches the company slug exactly
4. As last resort: the handle linked from `<DOMAIN>/about` (kick a quick `web-scraping` call only if the user explicitly asks)

#### Emails — only same-domain matches

Skip the greedy global email regex. Restrict to the company domain and surface generic inboxes separately:

```bash
# Only emails ending in @<DOMAIN>
jq -r '(.organicResults // [])[] | "\(.title) \(.snippet // "")"' \
  .hasdata/enrich/$slug-email-domain.json .hasdata/enrich/$slug-site-mention.json 2>/dev/null \
  | grep -Eoi "[A-Za-z0-9._%+-]+@${DOMAIN}" \
  | tr 'A-Z' 'a-z' | sort -u
```

Bucket the result:

- **Personal**: `firstname@`, `firstinitial@`, `firstname.lastname@` — high confidence if name tokens match
- **Generic inbox**: `info@`, `support@`, `hello@`, `contact@`, `sales@`, `careers@` — useful but not personal
- **Drop**: `noreply@`, `no-reply@`, `donotreply@`, third-party trackers

Do **not** infer pattern-based emails (e.g. "j@hasdata.com because Prospeo says 66.7%"). Only return what was actually surfaced verbatim.

#### Crunchbase + canonical site

```bash
jq -r '(.organicResults // [])[] | .link' ".hasdata/enrich/$slug-crunchbase.json" 2>/dev/null \
  | grep -Ei 'crunchbase\.com/(organization|person)/' | head -1

# Knowledge graph website (most authoritative)
jq -r '.knowledgeGraph.website // empty' ".hasdata/enrich/$slug-identity.json"
```

### 4a. Augmentation pass — scrape canonical pages (conditional)

SERP snippets are truncated and miss content behind JS. After the dork pass completes, scrape a small set of canonical URLs to pull verbatim emails, follower counts, and role text. **Conditional fires only** — keep this pass cheap.

Run scrapes in parallel, markdown output (LLM-friendly):

```bash
# (a) Canonical site — always run if DOMAIN is known and email count < 2 OR description is missing.
#     Cover root + the two most reliable contact pages.
if [ -n "$DOMAIN" ]; then
  for path in "" "/about" "/about-us" "/contact" "/team"; do
    url="https://${DOMAIN}${path}"
    out=".hasdata/enrich/$slug-site$(echo "$path" | tr '/' '-').md"
    hasdata web-scraping --url "$url" --output-format markdown --no-screenshot \
      -o "$out" 2>/dev/null &
  done
fi

# (b) X handle tie-breaker — only when 2+ candidate X handles survived dorks
#     and no inlineTwitter / KG follower count was found.
for handle_url in "${X_CANDIDATES[@]}"; do
  hasdata web-scraping --url "$handle_url" --output-format markdown --no-screenshot \
    -o ".hasdata/enrich/$slug-x-$(basename "$handle_url").md" 2>/dev/null &
done

# (c) LinkedIn personal tie-breaker — only when 2+ /in/ candidates survived
#     and the slug-token validator couldn't pick one.
for li_url in "${LI_CANDIDATES[@]}"; do
  hasdata web-scraping --url "$li_url" --output-format markdown --no-screenshot \
    -o ".hasdata/enrich/$slug-li-$(basename "$li_url").md" 2>/dev/null &
done

wait
```

**What to extract from the scrapes:**

```bash
# Verbatim emails on the company's own pages (highest confidence — owner-published)
grep -hEoi "[A-Za-z0-9._%+-]+@${DOMAIN}" .hasdata/enrich/$slug-site*.md 2>/dev/null \
  | tr 'A-Z' 'a-z' | sort -u

# Social links from the canonical site (footer / contact) — confirms canonical handles
grep -hEo 'https?://(linkedin\.com/(in|company)/[A-Za-z0-9._/-]+|github\.com/[A-Za-z0-9_-]+|(twitter|x)\.com/[A-Za-z0-9_]+|crunchbase\.com/[A-Za-z0-9._/-]+)' \
  .hasdata/enrich/$slug-site*.md 2>/dev/null | sort -u

# X follower count from a scraped profile page (look for "Followers" near a number)
for f in .hasdata/enrich/$slug-x-*.md; do
  echo "=== $f ==="
  grep -B1 -A1 -iE 'follower' "$f" | head -10
done

# LinkedIn role hint (look for a current title / company near the top of the page)
for f in .hasdata/enrich/$slug-li-*.md; do
  echo "=== $f ==="
  head -40 "$f"
done
```

**Merge rules:**

- An email surfaced by `site:` dork **and** the scrape of `<DOMAIN>/about|/contact` is high confidence — keep.
- A social link present in the canonical site's footer **wins** over a slug-only match from dorks — it's the owner's declared handle.
- For ambiguous X handles, pick the one with the higher scraped follower count. Drop the others silently.
- For ambiguous LinkedIn `/in/` profiles, pick the one whose scraped page mentions the company token in the current role. Drop the others silently.

If a scrape fails (site down, blocked, JS-only with no markdown), ignore it and proceed with what dorks gave us. Do not surface scrape failures in the output.

### 5. Drop noisy categories silently

- **Phone numbers**: do not extract via regex. Surface only if a phone appears inside `knowledgeGraph`.
- **Failed slug validation**: drop and never mention.
- **Pattern-inferred emails** (e.g. `j@hasdata.com` because some tool says 66.7% of emails follow that pattern): never include. Only verbatim hits.

### 6. Internal filter (do not narrate)

For each candidate signal, accept it if **any** of:
- It appears in `knowledgeGraph.website` / `knowledgeGraph.socialProfiles`
- It's surfaced by a `site:<DOMAIN>` dork
- Its slug contains a name or company token (after normalizing)
- An `inlineTwitter` block confirms it (with follower count)

If none, drop silently. **Do not output a "skipped" or "unverified" list.** Do not narrate the filtering process. Do not mention what was rejected, why the knowledge graph was missing, or what couldn't be confirmed.

### 7. Output — facts only

Use this exact template. **Omit any section with no surfaced data.** Do not add caveats, confidence labels, parenthetical attributions, cost notes, or follow-up offers.

```markdown
## <NAME or COMPANY>

**Domain:** <DOMAIN>
**Description:** <one short line — only if from knowledgeGraph.description or canonical site title>

**Emails**
- <email@DOMAIN> — personal
- <email@DOMAIN> — inbox

**Social**
- LinkedIn: <linkedin.com/in/...>
- LinkedIn (company): <linkedin.com/company/...>
- GitHub: <github.com/...>
- X: <@handle> · <N followers>
- Crunchbase: <crunchbase.com/...>

**Mentions**
- <url>
- <url>
- <url>
```

Rules for the output:
- No confidence labels (`high` / `medium` / `low`).
- No "skipped", "unverified", "caveats", or "couldn't confirm" sections.
- No "no knowledge graph rendered" notes.
- No inline parenthetical sourcing (`(from terms-of-use)`, `(slug match)`, etc.).
- No cost / credits line.
- No closing question or follow-up offer.
- If a section (Emails / Social / Mentions) has zero entries that passed the filter, omit the heading entirely.
- Cap Mentions at 5, deduped by domain.
- For X handles: include followers count only if surfaced from `inlineTwitter` or `knowledgeGraph` — otherwise just the handle.

That's the entire response. Nothing before, nothing after.

## Cost (for command author reference, not for output)

- `google-serp` dorks: 10 credits each. Person + company ≈ 9 dorks ≈ 90. Company-only ≈ 60. Name-only discovery (step 1a) ≈ 30 then stop.
- `web-scraping` augmentation: 10 credits per URL. Site pass ≈ 3–5 URLs (`/`, `/about`, `/contact`, `/team`) ≈ 30–50. X / LinkedIn tie-breakers fire only on ambiguity ≈ 0–30 extra.
- Typical full run (person + company, no ambiguity): ≈ 120–140 credits. With X/LinkedIn tie-breakers: ≈ 150–170.
