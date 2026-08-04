---
name: hasdata-realestate
description: |
  Real-estate, short-term-rental, and hotel data from Zillow, Redfin, Airbnb, and Booking.com. Use this skill when the user wants property listings, sold comps, rental searches, vacation rentals, hotel availability and prices, or full property details. Triggers on "Zillow listings in", "homes for sale in", "houses for rent in", "Redfin search", "sold comps for", "Airbnb in", "vacation rentals", "hotels in", "where to stay in", "hotel prices for", "book a room in", "Zillow property at <url>", "Airbnb listing <url>", "Booking.com hotel <url>", or any real-estate, short-term rental, or accommodation request. Returns structured JSON — addresses, prices, beds/baths, square footage, photos, agent info, room availability — without HTML scraping. Supports rich filters (price range, beds, lot size, days on market, HOA, star rating, facilities, etc.).
allowed-tools:
  - Bash(hasdata *)
---

# hasdata real-estate and accommodation APIs

Listings and full property details from Zillow, Redfin, Airbnb, and Booking.com.

## When to use

- User wants for-sale, for-rent, or sold listings in a city/area
- User wants full details on a single Zillow / Redfin / Airbnb / Booking.com URL
- User wants to filter by price, beds, baths, square footage, lot size, HOA, etc.
- User wants short-term rentals (Airbnb) for specific dates and party size
- User wants hotels for specific stay dates, with prices and room availability

## APIs in this group

| Command            | Purpose                                                    | Cost |
| ------------------ | ---------------------------------------------------------- | ---- |
| `zillow-listing`   | Zillow search — for-sale / for-rent / sold by location     | 5    |
| `zillow-property`  | Full Zillow property details by URL                        | 5    |
| `redfin-listing`   | Redfin search by zipcode with rich filters                 | 5    |
| `redfin-property`  | Full Redfin property details by URL                        | 5    |
| `airbnb-listing`   | Airbnb search by location, dates, party size               | 5    |
| `airbnb-property`  | Full Airbnb listing details by URL                         | 5    |
| `booking-search`   | Booking.com search by destination and stay dates           | 10   |
| `booking-place`    | Full Booking.com property details plus available rooms     | 10   |

## Quick start

### Zillow

```bash
# For-sale homes in NYC, $500k–$1M, 2+ beds
hasdata zillow-listing --keyword "New York, NY" --type forSale \
  --price-min 500000 --price-max 1000000 --beds-min 2 \
  --pretty -o .hasdata/zillow-nyc-sale.json

# For-rent
hasdata zillow-listing --keyword "Brooklyn, NY" --type forRent --pretty -o .hasdata/zillow-bk-rent.json

# Sold (comps)
hasdata zillow-listing --keyword "Austin, TX" --type sold --days-on-zillow "12m" --pretty -o .hasdata/zillow-atx-sold.json

# Full property details
hasdata zillow-property --url "https://www.zillow.com/homedetails/.../31543731_zpid/" --pretty -o .hasdata/zillow-prop.json

# With agent emails (slightly higher cost)
hasdata zillow-property --url "https://www.zillow.com/homedetails/..." --extract-agent-emails --pretty -o .hasdata/zillow-prop-emails.json
```

### Redfin

```bash
# For-sale, 3+ beds, 2+ baths. Redfin searches by zipcode, not city name.
hasdata redfin-listing --keyword "98101" --type forSale --beds-min 3 --baths "two" \
  --pretty -o .hasdata/redfin-seattle.json

# Full property
hasdata redfin-property --url "https://www.redfin.com/IL/Chicago/.../home/12694628" --pretty -o .hasdata/redfin-prop.json
```

### Airbnb

```bash
# Search by location, dates, guests
hasdata airbnb-listing --location "Lisbon, Portugal" \
  --check-in "2026-06-01" --check-out "2026-06-08" --adults 2 \
  --pretty -o .hasdata/airbnb-lisbon.json

# Full listing details
hasdata airbnb-property --url "https://www.airbnb.com/rooms/7777642" --pretty -o .hasdata/airbnb-prop.json

# Pagination
hasdata airbnb-listing --location "Paris" --check-in "2026-06-01" --check-out "2026-06-05" \
  --next-page-token "<token>" --pretty -o .hasdata/airbnb-paris-p2.json
```

### Booking.com

```bash
# Hotels in Paris for two adults, no children
hasdata booking-search --keyword "Paris" \
  --check-in-date "2026-09-10" --check-out-date "2026-09-14" \
  --adults 2 --children 0 --rooms 1 \
  --pretty -o .hasdata/booking-paris.json

# Full property details plus available rooms for those dates
hasdata booking-place --url "https://www.booking.com/hotel/fr/le-bristol-paris.html" \
  --check-in-date "2026-09-10" --check-out-date "2026-09-14" \
  --adults 2 --children 0 --rooms 1 \
  --pretty -o .hasdata/booking-bristol.json
```

## Common Booking.com flags

| Flag                     | Purpose                                                              |
| ------------------------ | -------------------------------------------------------------------- |
| `--keyword <query>`      | Required for search — city, region, neighborhood, or property name   |
| `--url <url>`            | Required for `booking-place` — full booking.com property URL         |
| `--check-in-date`        | Required — `YYYY-MM-DD`, must be in the future                        |
| `--check-out-date`       | Required — `YYYY-MM-DD`, later than check-in                          |
| `--adults <n>`           | Required — adult guests across all rooms                              |
| `--children <n>`         | Required — pass `0`. Non-zero is broken in the CLI, see Tips           |
| `--rooms <n>`            | Required — number of rooms to book                                    |
| `--currency`             | Response currency, or `hotelCurrency` to keep each property's own     |
| `--facilities`           | Filter by property facilities, values combined with OR               |
| `--bedrooms` / `--bathrooms` | Minimum counts                                                    |

## Common Zillow flags

| Flag                       | Purpose                                                                  |
| -------------------------- | ------------------------------------------------------------------------ |
| `--keyword <location>`     | Required — location string (e.g., `"Brooklyn, NY"`)                      |
| `--type`                   | Required — `forSale`, `forRent`, `sold`                                  |
| `--price-min` / `--price-max`  | Price range                                                          |
| `--beds-min` / `--beds-max`    | Beds range                                                           |
| `--baths-min` / `--baths-max`  | Baths range                                                          |
| `--square-feet-min` / `--square-feet-max` | Sqft range                                                |
| `--lot-size-min` / `--lot-size-max` | Lot size range                                                  |
| `--home-types`             | Array (e.g. `Houses`, `Townhomes`, `Condos`)                             |
| `--days-on-zillow`         | `6m`, `12m`, `24m`, `36m`                                                |
| `--hoa <max>`              | HOA fee cap                                                              |
| `--keywords`               | Free-form keyword (e.g. `"renovated, garage"`)                           |
| `--sort`                   | `homesForYou`, `priceHighToLow`, `priceLowToHigh`, `newest`, `bedrooms`, … |
| `--page <n>`               | Pagination                                                               |
| `--pets`                   | Array of pet options                                                     |
| `--must-have-garage`       | Filter to garage-only                                                    |

## Common Airbnb flags

| Flag                  | Purpose                                                            |
| --------------------- | ------------------------------------------------------------------ |
| `--location`          | Required — location (e.g., `"Paris, France"`)                      |
| `--check-in`          | Required — `YYYY-MM-DD`                                            |
| `--check-out`         | `YYYY-MM-DD`                                                       |
| `--adults` / `--children` / `--infants` / `--pets` | Party composition                     |
| `--next-page-token`   | Pagination token from previous response                            |

## Tips

- **Booking.com searches with children do not work in CLI v0.2.1.** `--children` is required
  and must be `0`. Any non-zero value fails with `HTTP 422 childrenAges requiredWhen`, because
  the CLI sends `childrenAgesJson` while the API expects `childrenAges` as a comma-separated
  list. For a family stay, call the API directly until the CLI is fixed:
  `https://api.hasdata.com/scrape/booking/search?...&children=2&childrenAges=3,7`
- **Booking calls cost 10 credits**, double the Zillow / Redfin / Airbnb calls, so filter
  before fanning out.
- **Airbnb vs Booking.com:** Airbnb for whole-home and short-term rentals, Booking.com for
  hotels and room-level availability. Query both when the user just says "where to stay".
- **Search → property fan-out** is the standard pattern for getting full details on multiple listings:
  ```bash
  hasdata zillow-listing --keyword "Austin, TX" --type forSale --pretty -o .hasdata/zillow.json
  for url in $(jq -r '.properties[].detailUrl' .hasdata/zillow.json | head -10); do
    hasdata zillow-property --url "$url" --pretty -o ".hasdata/zillow-$(basename $url).json" &
  done
  wait
  ```
- For **comps**, use `zillow-listing --type sold --days-on-zillow 6m` (recently sold) — the property API returns full sold history per home.
- **Airbnb dates are required** — without `--check-in`, search returns very limited data.
- `--extract-agent-emails` on Zillow property is a paid upgrade — only enable for lead-gen workflows.
- Both Redfin and Zillow have rich filter sets (40+ flags). Run `hasdata zillow-listing --help` / `hasdata redfin-listing --help` for the full list.

## Working with results

```bash
# Zillow listing → address, price, beds/baths, URL
jq -r '.properties[] | "\(.address)\t$\(.price)\t\(.beds)bd/\(.baths)ba\t\(.detailUrl)"' .hasdata/zillow.json

# Redfin → similar
jq -r '.listings[] | "\(.address)\t$\(.price)\t\(.beds)bd/\(.baths)ba"' .hasdata/redfin.json

# Airbnb → name, price/night, rating, link
jq -r '.listings[] | "\(.title)\t$\(.price.total // .price.amount)\t\(.rating // "-")★\t\(.url)"' .hasdata/airbnb.json
```

## See also

- [hasdata-maps](../hasdata-maps/SKILL.md) — Google Maps for non-listing local-business data
- [hasdata-scrape](../hasdata-scrape/SKILL.md) — for non-Zillow/Redfin/Airbnb real-estate sites
