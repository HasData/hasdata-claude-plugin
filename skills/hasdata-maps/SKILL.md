---
name: hasdata-maps
description: |
  Google Maps data — find local businesses by query and location, pull a place's full profile (address, phone, website, hours, ratings), get reviews, photos, and the posts a business publishes on its listing. Use this skill when the user wants local business info, says "find restaurants in", "Google Maps for X near Y", "get reviews for this place", "phone number for", "address of", "businesses near me", "what's open in", "photos of this restaurant", "any offers from this place", "what has this business posted", or otherwise needs structured local-business data. Returns JSON with names, addresses, phones, websites, ratings, GPS coordinates — no scraping needed. Use this instead of generic web-scraping for Google Maps URLs.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata Google Maps APIs

Structured Google Maps data: place search, place details, reviews, photos.

## When to use

- The user wants local business info (name, address, phone, website, hours, rating)
- They want reviews or photos for a known place
- They mention "near me", a city + business type, or a place ID
- They want lead-gen / contact data for businesses on Maps

For non-Maps "what does X website look like" / general web search, use [hasdata-search](../hasdata-search/SKILL.md) or [hasdata-scrape](../hasdata-scrape/SKILL.md).

## APIs in this group

| Command                              | Purpose                                                             | Cost |
| ------------------------------------ | ------------------------------------------------------------------- | ---- |
| `google-maps`                        | Search places by query + location/coords                            | 5    |
| `google-maps-place`                  | Full profile of a single place (by place ID)                        | 5    |
| `google-maps-reviews`                | Reviews for a place (by place ID or data ID)                        | 5    |
| `google-maps-contributor-reviews`    | All reviews by a Google contributor                                 | 5    |
| `google-maps-photos`                 | Photos for a place                                                  | 5    |
| `google-maps-posts`                  | Posts a business published on its listing (offers, events, news)    | 10   |

## Quick start

```bash
# Search "pizza near Brooklyn"
hasdata google-maps --q "pizza" --ll "@40.6782,-73.9442,14z" --pretty -o .hasdata/maps-pizza-bk.json

# Search by canonical location
hasdata google-maps --q "yoga studios" --ll "@40.7455,-74.0083,14z" --pretty -o .hasdata/maps-yoga-nyc.json

# Get full place profile
hasdata google-maps-place --place-id "ChIJFU2bda4SM4cRKSCRyb6pOB8" --pretty -o .hasdata/place.json

# Reviews, sorted newest first
hasdata google-maps-reviews --place-id "ChIJFU2bda4SM4cRKSCRyb6pOB8" --sort-by newestFirst --pretty -o .hasdata/reviews.json

# Photos for a place
hasdata google-maps-photos --place-id "ChIJFU2bda4SM4cRKSCRyb6pOB8" --pretty -o .hasdata/photos.json

# Posts a business published on its listing (offers, events, announcements)
hasdata google-maps-posts --place-id "ChIJFU2bda4SM4cRKSCRyb6pOB8" --pretty -o .hasdata/posts.json

# Pagination — use the next-page-token from the previous response
hasdata google-maps-reviews --place-id "ChIJ..." --next-page-token "<token>" --pretty -o .hasdata/reviews-p2.json
```

## google-maps key flags

| Flag             | Purpose                                                                                |
| ---------------- | -------------------------------------------------------------------------------------- |
| `--q <query>`    | Search term (required)                                                                 |
| `--ll <coords>`  | GPS center + zoom: `@lat,lng,Nz` (default `@40.7455,-74.0083,14z`)                     |
| `--start <n>`    | Pagination offset — multiples of 20 (0 = page 1, 20 = page 2)                          |
| `--gl <code>`    | Country code                                                                           |
| `--hl <code>`    | UI language                                                                            |
| `--domain`       | Google domain (e.g. `google.co.uk`)                                                    |

`--ll` is the most important flag — narrow searches with tighter zoom (`14z`–`16z`) for neighborhood-level results, wider (`10z`–`12z`) for citywide.

## google-maps-reviews key flags

| Flag                    | Purpose                                                          |
| ----------------------- | ---------------------------------------------------------------- |
| `--place-id <id>`       | Place ID (one of place-id or data-id required)                   |
| `--data-id <id>`        | Maps data ID (alternative to place-id)                           |
| `--sort-by`             | `qualityScore`, `newestFirst`, `ratingHigh`, `ratingLow`         |
| `--topic-id <id>`       | Filter by review topic (e.g., service, food)                     |
| `--next-page-token`     | Pagination token from previous response                          |

## Tips

- **Place IDs** (start with `ChIJ`) are stable and preferred for `place`, `reviews`, `photos`. Get them from `google-maps` results (`.places[].placeId`).
- **GPS coordinates** for `--ll`: pull from any maps URL or use `--gl` + a city name in `--q`.
- For **lead gen** (extracting contact info for many businesses), search → fan out:
  ```bash
  hasdata google-maps --q "dental clinics" --ll "@40.7,-74.0,12z" --pretty -o .hasdata/dentists.json
  for pid in $(jq -r '.places[].placeId' .hasdata/dentists.json); do
    hasdata google-maps-place --place-id "$pid" --pretty -o ".hasdata/dentist-$pid.json" &
  done
  wait
  ```
- For **review monitoring**, paginate with `--next-page-token` and sort `newestFirst`.
- **`google-maps-posts`** takes either `--place-id` or `--data-id`, and costs 10 credits rather
  than 5. Useful for tracking a competitor's promotions and events over time.

## Working with results

```bash
# Place search → name, rating, phone, website
jq -r '.places[] | "\(.title)\t\(.rating)★\t\(.phone // "n/a")\t\(.website // "n/a")"' .hasdata/maps-pizza.json

# Reviews → user, rating, text
jq -r '.reviews[] | "[\(.rating)★] \(.user.name): \(.snippet // .text // "")"' .hasdata/reviews.json

# Photos → image URLs
jq -r '.photos[].image' .hasdata/photos.json
```

## See also

- [hasdata-business](../hasdata-business/SKILL.md) — Yelp / YellowPages for non-Google business directories
- [hasdata-search](../hasdata-search/SKILL.md) — broader web search (not just Maps)
- [hasdata-scrape](../hasdata-scrape/SKILL.md) — for any non-Maps URL
