---
name: hasdata-hotels
description: |
  Hotel prices and availability from Google Hotels and Booking.com. Use this skill when the user wants hotels in a city, a room for specific dates, or details for a Booking.com property. Triggers on "hotels in", "where to stay in", "hotel prices for", "book a room in", "Booking.com". Returns structured JSON with prices and availability. Ask for check-in and check-out dates when the user did not give them. For homes, Zillow, Redfin, or Airbnb, use the real-estate skill.
allowed-tools:
  - Bash(hasdata *)
---

# Hotels

## How to fetch

1. If a HasData tool is connected, call the one whose name matches the task below. Read that tool's schema and fill the fields it asks for. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector for this chat. Do not install software and do not ask them to paste an API key.
3. Use the `hasdata` command only when the connector cannot be connected and the binary is already on PATH. Booking.com commands are `booking-search` and `booking-place`. A Google Hotels CLI command is usable only if `hasdata --help` lists it.

## Which tool

| Ask | Tool name contains | Required |
| --- | --- | --- |
| Hotels on Google for a place and dates | `google_travel_hotels` | `q`, `checkInDate`, `checkOutDate` |
| Hotels on Booking.com | `booking_search` | `keyword`, `checkInDate`, `checkOutDate`, `rooms`, `adults`, `children` |
| One Booking.com property and its rooms | `booking_place` | `url`, `checkInDate`, `checkOutDate`, `rooms`, `adults`, `children` |

Dates are `YYYY-MM-DD`. If the user did not give dates, guest count, or room count, ask before calling. Do not invent a default stay.

For homes for sale, rent, Zillow, Redfin, or Airbnb, use [hasdata-realestate](../hasdata-realestate/SKILL.md). For flights, use [hasdata-flights](../hasdata-flights/SKILL.md).

Return property name, price, rating, and link from the tool output.
