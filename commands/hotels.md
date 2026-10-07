---
description: Hotel prices for a city and stay dates
argument-hint: city, check-in, check-out
allowed-tools:
  - Bash(hasdata *)
---

# /hasdata:hotels

Stay: **$ARGUMENTS**

You need a place, a check-in date, and a check-out date. If any of the three is missing, ask before calling a tool. Dates are YYYY-MM-DD. Do not invent a stay.

If HasData tools are connected, call the tool whose name contains `google_travel_hotels` (`q`, `checkInDate`, `checkOutDate`). Also call the tool whose name contains `booking_search` when the user asked for Booking.com or for more than one site (`keyword`, `checkInDate`, `checkOutDate`, `rooms`, `adults`, `children`). Read each schema. Do not run a shell command when the tools exist.

If no HasData tool is connected, ask the user to press Connect. Do not install software and do not ask for an API key.

Return property name, price, rating, and link from the tool output.
