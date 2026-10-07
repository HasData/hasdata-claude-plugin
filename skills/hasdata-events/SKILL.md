---
name: hasdata-events
description: |
  Events Google shows for a query, then the details of one event. Use this skill when the user wants concerts, conferences, or other events from Google, or the full card for one event in those results. Triggers on "events in", "Google events for", "concerts in", "what's on in". The event list comes from the full Google results page. One event's details need the page token from that list, and the token expires in a few minutes.
allowed-tools:
  - Bash(hasdata *)
---

# Google events

## How to fetch

1. If HasData tools are connected, call the full results tool whose name contains `google_serp_serp` and does not contain `light`. Pass the query in `q`, plus `gl`, `hl`, and `location` when the user names them.
2. Read `eventsResults` in that response. List title, date, and venue from those entries.
3. When the user wants one event, immediately call the tool whose name contains `google_serp_events` with the `pageToken` from that entry. The token expires in a few minutes. The events tool does not take a city name.

If `eventsResults` is missing, say Google returned no events for that query. Do not invent an event.

If no HasData tool is connected, ask the user to press Connect. Do not install software and do not ask for an API key.
