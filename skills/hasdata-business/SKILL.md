---
name: hasdata-business
description: |
  Local business directory data from Yelp and YellowPages — search, full place profiles, ratings, reviews, contact info. Use this skill when the user wants Yelp listings, YellowPages results, restaurant reviews on Yelp, plumbers / contractors / professionals near a city, business phone numbers and addresses, or full Yelp/YellowPages place profiles. Triggers on "Yelp search for", "Yelp reviews for", "find <category> on Yelp", "YellowPages for", "plumbers in <city>", "lawyers near", "lead list of <category>", "phone number for". Returns structured JSON. Use this when the user names Yelp or YellowPages specifically; for Google Maps data use hasdata-maps.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata business APIs

Search and place profiles for Yelp and YellowPages.

## When to use

- User explicitly mentions Yelp or YellowPages
- User wants a non-Google directory (Google Maps → use [hasdata-maps](../hasdata-maps/SKILL.md))
- User wants lead lists of a service category in a city (plumbers, lawyers, dentists, etc.)
- User wants Yelp reviews / ratings for a known place

## APIs in this group

| Command                | Purpose                                                  | Cost |
| ---------------------- | -------------------------------------------------------- | ---- |
| `yelp-search`          | Yelp search by keyword + location                        | 5    |
| `yelp-place`           | Full Yelp place profile (by Yelp ID or alias)            | 5    |
| `yellowpages-search`   | YellowPages search by keyword + location                 | 5    |
| `yellowpages-place`    | Full YellowPages place profile (by URL)                  | 5    |

## Quick start

### Yelp

```bash
# Search
hasdata yelp-search --keyword "ramen" --location "Brooklyn, NY" --pretty -o .hasdata/yelp-ramen.json

# Different country
hasdata yelp-search --keyword "fish and chips" --location "London" --domain www.yelp.co.uk --pretty -o .hasdata/yelp-uk.json

# Map area filter (lat/lng bounding box)
hasdata yelp-search --keyword "tacos" --location "Houston, TX" \
  --l "g:-95.2486,29.8496,-95.4277,29.6324" --pretty -o .hasdata/yelp-tacos.json

# Full place profile (use Yelp alias or Yelp ID)
hasdata yelp-place --place-id "mcdonalds-new-york-386" --pretty -o .hasdata/yelp-place.json
hasdata yelp-place --place-id "jPIZ3FR5LNcwPuUHi2Fe4g" --pretty -o .hasdata/yelp-place-by-id.json
```

### YellowPages

```bash
# Search
hasdata yellowpages-search --keyword "Plumbers" --location "New York, NY" --pretty -o .hasdata/yp-plumbers.json

# Sort by rating
hasdata yellowpages-search --keyword "Lawyers" --location "Austin, TX" --sort averageRating --pretty -o .hasdata/yp-lawyers.json

# Canada
hasdata yellowpages-search --keyword "Dentists" --location "Toronto, ON" --domain www.yellowpages.ca --pretty -o .hasdata/yp-dentists-ca.json

# Full place profile by URL
hasdata yellowpages-place --url "https://www.yellowpages.com/kings-county-ny/mip/aladdin-plumbing-corp-548289617" --pretty -o .hasdata/yp-place.json
```

## Common flags

### Yelp
| Flag             | Purpose                                                                  |
| ---------------- | ------------------------------------------------------------------------ |
| `--keyword`      | Required — query (e.g., `"sushi"`)                                       |
| `--location`     | Required — `"City, ST"`                                                  |
| `--start <n>`    | Pagination offset (multiples of 10)                                      |
| `--l <bbox>`     | Map area: `g:lng1,lat1,lng2,lat2`                                        |
| `--domain`       | Yelp domain (40+ supported)                                              |

### YellowPages
| Flag             | Purpose                                                                  |
| ---------------- | ------------------------------------------------------------------------ |
| `--keyword`      | Required — category (e.g., `"Electricians"`)                             |
| `--location`     | Required — `"City, ST"`                                                  |
| `--sort`         | `default`, `distance`, `averageRating`, `name`                           |
| `--page <n>`     | Pagination                                                               |
| `--domain`       | `www.yellowpages.com` or `www.yellowpages.ca`                            |

## Tips

- **Lead lists** — search → place fan-out gets you full contact info per business:
  ```bash
  hasdata yellowpages-search --keyword "Plumbers" --location "Brooklyn, NY" --pretty -o .hasdata/plumbers.json
  for url in $(jq -r '.places[].url' .hasdata/plumbers.json | head -20); do
    hasdata yellowpages-place --url "$url" --pretty -o ".hasdata/plumber-$(basename $url).json" &
  done
  wait
  ```
- **Yelp aliases** (e.g. `mcdonalds-new-york-386`) are stable and human-readable; **Yelp IDs** are opaque. Either works in `--place-id`.
- For **Google's local business data**, use [hasdata-maps](../hasdata-maps/SKILL.md) — it's typically more comprehensive than Yelp for large markets.
- YellowPages is best for **service trades** (plumbers, electricians, contractors) — it has stronger contact-info coverage.

## Working with results

```bash
# Yelp search → name, rating, phone, address
jq -r '.places[] | "\(.title)\t\(.rating)★\t\(.phone // "n/a")\t\(.address // "n/a")"' .hasdata/yelp.json

# YellowPages → name, phone, address, rating
jq -r '.places[] | "\(.title)\t\(.phone // "n/a")\t\(.address // "n/a")\t\(.rating // "n/a")"' .hasdata/yp.json

# Filter for high-rated only
jq '.places[] | select((.rating // 0) >= 4.5)' .hasdata/yp.json
```

## See also

- [hasdata-maps](../hasdata-maps/SKILL.md) — Google Maps for the same kind of local data, often more comprehensive
- [hasdata-search](../hasdata-search/SKILL.md) — `google-serp` for general business research
