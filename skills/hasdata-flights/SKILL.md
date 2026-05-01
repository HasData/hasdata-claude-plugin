---
name: hasdata-flights
description: |
  Real-time flight search via Google Flights — one-way, round-trip, or multi-city, with rich filters (airlines, stops, layover duration, travel class, max price, lower emissions). Use this skill when the user wants flight prices, schedules, or to compare itineraries. Triggers on "flight prices from X to Y", "Google Flights for", "cheapest flights", "round-trip from", "non-stop flights", "business class flights", or any travel-fare research. Returns structured JSON — airlines, prices, durations, stops, carbon emissions — without HTML scraping.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata Google Flights API

Real-time flight search via Google Flights.

## When to use

- User wants flight prices for specific dates and routes
- They want to compare airlines, stops, or travel classes
- They want to filter by max price, max duration, layover length, or emissions
- They want round-trip, one-way, or multi-city itineraries

## API in this group

| Command           | Purpose                                             | Cost |
| ----------------- | --------------------------------------------------- | ---- |
| `google-flights`  | Search Google Flights with full filter support      | 15   |

## Quick start

```bash
# One-way SFO → JFK on a specific date
hasdata google-flights \
  --departure-id "SFO" --arrival-id "JFK" \
  --outbound-date "2026-06-15" \
  --type "oneWay" \
  --pretty -o .hasdata/sfo-jfk.json

# Round-trip with return
hasdata google-flights \
  --departure-id "JFK" --arrival-id "LHR" \
  --outbound-date "2026-07-10" --return-date "2026-07-20" \
  --type "roundTrip" \
  --adults 2 \
  --pretty -o .hasdata/jfk-lhr-rt.json

# Non-stop, max $800, sorted by price
hasdata google-flights \
  --departure-id "LAX" --arrival-id "NRT" \
  --outbound-date "2026-09-05" --return-date "2026-09-15" \
  --stops "0" --max-price 800 --sort-by "price" \
  --pretty -o .hasdata/lax-nrt-nonstop.json

# Business class, prefer fewer emissions
hasdata google-flights \
  --departure-id "JFK" --arrival-id "CDG" \
  --outbound-date "2026-08-01" --return-date "2026-08-08" \
  --travel-class "Business" --less-emissions \
  --pretty -o .hasdata/jfk-cdg-biz.json

# Specific airlines only
hasdata google-flights \
  --departure-id "SEA" --arrival-id "HND" \
  --outbound-date "2026-10-01" \
  --include-airlines "DL,JL" \
  --pretty -o .hasdata/sea-hnd-dljl.json
```

## Common flags

| Flag                          | Purpose                                                                       |
| ----------------------------- | ----------------------------------------------------------------------------- |
| `--departure-id <code>`       | IATA airport code or location kgmid (e.g., `SFO`, `/m/0d6lp`)                 |
| `--arrival-id <code>`         | IATA airport code or location kgmid                                           |
| `--outbound-date YYYY-MM-DD`  | Outbound date (required)                                                      |
| `--return-date YYYY-MM-DD`    | Return date (required for `roundTrip`)                                        |
| `--type`                      | `oneWay`, `roundTrip`, `multiCity`                                            |
| `--adults <n>`                | Adult passengers                                                              |
| `--children` / `--bags` / `--infants-in-seat` / `--infants-on-lap` | Party composition         |
| `--travel-class`              | `Economy`, `Premium Economy`, `Business`, `First`                             |
| `--stops`                     | Restrict stops (`0` = nonstop, `1`, `2`)                                      |
| `--currency <code>`           | Result currency (default `USD`)                                               |
| `--max-price <n>`             | Max price (in selected currency)                                              |
| `--max-duration <minutes>`    | Max total flight duration                                                     |
| `--layover-duration "min,max"`| Layover range in minutes (e.g. `45,180`)                                      |
| `--include-airlines <codes>`  | Comma-separated IATA airline codes (e.g., `UA,DL`)                            |
| `--exclude-airlines <codes>`  | Exclude airlines                                                              |
| `--include-connections <codes>` | Allowed connection airports                                                 |
| `--exclude-connections <codes>` | Forbidden connection airports                                               |
| `--outbound-times "from,to"`  | Departure window (hours, e.g., `6,20`)                                        |
| `--return-times "from,to"`    | Return-leg time window                                                        |
| `--less-emissions`            | Prefer flights with lower CO₂                                                 |
| `--sort-by`                   | `price`, `departureTime`, `arrivalTime`, `duration`, `emissions`              |
| `--deep-search`               | Slower but matches what google.com/flights shows                              |
| `--show-hidden`               | Include hidden / cheaper-when-booked-as-pieces fares                          |
| `--departure-token <token>`   | Select a specific outbound to get matching returns                            |
| `--booking-token <token>`     | Get final booking options for a selected itinerary                            |
| `--multi-city-json <json>`    | Multi-city JSON (array of leg objects with `departureId`, `arrivalId`, `date`)|

## Tips

- **Round-trip selection is two-step**: the first call returns outbound options each with a `departureToken`. To get matching return flights, call again with `--departure-token "<token>"`. To finalize and see booking options, call with `--booking-token`.
- **`--deep-search`** reproduces what you see in a browser at google.com/flights but takes longer. Use only when initial results seem incomplete.
- For **price tracking** (e.g. monitor a route over weeks), schedule the same query and `jq` the cheapest result:
  ```bash
  jq '.bestFlights | min_by(.price)' .hasdata/route.json
  ```
- **Carbon-aware travel**: combine `--less-emissions --sort-by emissions` for the lowest-CO₂ options.

## Working with results

```bash
# All flights → airline, price, duration, stops
jq -r '.bestFlights[] | "\(.airlines[0])\t$\(.price)\t\(.totalDuration)min\t\(.layovers | length) stops"' .hasdata/flights.json

# Cheapest non-stop
jq '[.bestFlights[] | select((.layovers | length) == 0)] | min_by(.price)' .hasdata/flights.json

# Sort by emissions
jq '.bestFlights | sort_by(.carbonEmissions.thisFlight)' .hasdata/flights.json
```

## See also

- [hasdata-search](../hasdata-search/SKILL.md) — `google-events` for non-flight travel events
- [hasdata-realestate](../hasdata-realestate/SKILL.md) — `airbnb-listing` for accommodation
