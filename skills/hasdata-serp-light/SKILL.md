---
name: hasdata-serp-light
description: |
  Fast Google organic results for enrichment and lookups: titles, links, and snippets, without the full page of ads and panels. Use this skill when the user wants the official site, a LinkedIn or Crunchbase URL, or a quick "search Google for" and does not need ads, the local pack, or an AI Overview token. Triggers on "find the official site", "Google the company", "search Google for", "who is", "enrich this company from Google". Country, language, and city still apply. The full SEO results page is hasdata-serp.
allowed-tools:
  - Bash(hasdata *)
---

# Google SERP light

## How to fetch

1. If a HasData tool is connected whose name contains `google_serp_serp_light`, call it. Read its schema. `q` is required. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector. Do not install software and do not ask for an API key.
3. Use `hasdata google-serp-light` only when the connector cannot be connected and the binary is already on PATH.

Pass `gl`, `hl`, `location`, `domain`, `num`, `start`, and `tbs` when the user named a country, language, city, page, or time window. `num` is 10 to 100. Do not invent `uule`.

This tool does not return a device split, the ads block, or an AI Overview token. If the user needs those, switch to [hasdata-serp](../hasdata-serp/SKILL.md) and call the full SERP tool instead.

Return title, link, and snippet. For a company lookup, point at the result that is the official site and say why (domain match or knowledge-panel style evidence in the response). Do not guess a URL the tool did not return.
