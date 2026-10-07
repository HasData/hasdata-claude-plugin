---
description: Lead list for a business category in a city, with name, phone, address, website, and rating
argument-hint: category in city, optional count such as n=30
allowed-tools:
  - Bash(hasdata *)
---

# /hasdata:leads

Query: **$ARGUMENTS**

Need a business category and a city. Optional `n=30` is how many leads to return after deduping. If either the category or the city is missing, ask. Default count is 30. Cap at 100.

## Connector

If HasData tools are connected, do only this and stop. Do not run a shell command and do not tell the user a file was written.

1. Call the tool whose name contains `google_maps_search`. Pass `q` as the category plus the city. If you know the city center, pass `ll` as `@lat,lng,14z`. Page with `start` at 0, then 20, then 40. Stop at 5 pages or when you have enough rows.
2. Call the tool whose name contains `yellowpages_search`. Pass `keyword` and `location`. Page with `page`.
3. Merge on name and phone. Drop duplicates.
4. Reply with a table: name, phone, address, website, rating. Use only values the tools returned. If a phone is missing, leave it blank.

If no HasData tool is connected, ask the user to press Connect. In Claude Code, authenticate with `/mcp`. Do not install a binary and do not ask for an API key.

## CLI fallback

Only when the connector cannot be connected and `hasdata` is already on PATH. Then `hasdata google-maps` and `hasdata yellowpages-search` with the same query. Do not install the CLI to reach this section.
