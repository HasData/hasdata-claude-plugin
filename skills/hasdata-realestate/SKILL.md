---
name: hasdata-realestate
description: |
  Homes and short-term rentals from Zillow, Redfin, and Airbnb. Use this skill for listings, sold comps, houses for rent, or one property URL. Triggers on "Zillow listings in", "homes for sale in", "houses for rent in", "Redfin search", "sold comps for", "Airbnb in", "vacation rentals". Hotels, Booking.com, and "where to stay" are hasdata-hotels, not this skill.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata real-estate APIs

Listings and property details from Zillow, Redfin, and Airbnb. Hotels and Booking.com are [hasdata-hotels](../hasdata-hotels/SKILL.md).

## How to fetch

1. If a HasData tool is connected, call the one in the table. Read its schema. Do not run a shell command, and skip the CLI fallback, while that tool exists. Do not call `api.hasdata.com` yourself.
2. If no HasData tool is connected, ask the user to press Connect. In Claude Code, authenticate with `/mcp`. Do not install a binary and do not ask for an API key.
3. The CLI fallback is only when the connector cannot be connected and `hasdata` is already on PATH.

| Ask | Tool name contains | Pass |
| --- | --- | --- |
| Zillow search | `zillow_listing` | `keyword` and `type` (`forSale`, `forRent`, or `sold`) |
| One Zillow home | `zillow_property` | `url` |
| Redfin search | `redfin_listing` | `keyword` (a zip code) and `type` |
| One Redfin home | `redfin_property` | `url` |
| Airbnb search | `airbnb_listing` | `location`, `checkIn`, `checkOut`. Ask for dates if they are missing |
| One Airbnb listing | `airbnb_property` | `url` |

Do not set `extractAgentEmails` unless the user asked for the listing agent's public contact. Hotels are not this skill.

## When to use

- User wants for-sale, for-rent, or sold listings in a city or zip code
- User wants full details on a Zillow, Redfin, or Airbnb URL
- User wants to filter by price, beds, baths, square footage, lot size, or HOA
- User wants an Airbnb for specific dates and a party size

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

## CLI fallback

Only when the connector cannot be connected and `hasdata` is already on PATH. Otherwise ignore this section, including the Booking.com examples. Hotels are hasdata-hotels.

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

Stay dates are placeholders. Both Airbnb and Booking.com reject dates in the past,
so resolve `<CHECK_IN>` / `<CHECK_OUT>` against today's date before running a
command — never copy a literal date from this file.

```bash
# Search by location, dates, guests
hasdata airbnb-listing --location "Lisbon, Portugal" \
  --check-in "<CHECK_IN>" --check-out "<CHECK_OUT>" --adults 2 \
  --pretty -o .hasdata/airbnb-lisbon.json

# Full listing details
hasdata airbnb-property --url "https://www.airbnb.com/rooms/7777642" --pretty -o .hasdata/airbnb-prop.json

# Pagination
hasdata airbnb-listing --location "Paris" --check-in "<CHECK_IN>" --check-out "<CHECK_OUT>" \
  --next-page-token "<token>" --pretty -o .hasdata/airbnb-paris-p2.json
```

### Booking.com

```bash
# Hotels in Paris for two adults, no children
hasdata booking-search --keyword "Paris" \
  --check-in-date "<CHECK_IN>" --check-out-date "<CHECK_OUT>" \
  --adults 2 --children 0 --rooms 1 \
  --pretty -o .hasdata/booking-paris.json

# Full property details plus available rooms for those dates
hasdata booking-place --url "https://www.booking.com/hotel/fr/le-bristol-paris.html" \
  --check-in-date "<CHECK_IN>" --check-out-date "<CHECK_OUT>" \
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

- Hotels and Booking.com are [hasdata-hotels](../hasdata-hotels/SKILL.md). Do not call `api.hasdata.com` directly and do not send the user there to paste a key.
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
