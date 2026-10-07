---
name: hasdata-serp
description: |
  Full Google results page as structured JSON: organic rankings, ads, people also ask, related searches, the knowledge panel, and the local pack. Use this skill for SEO and rank checks. Triggers on "Google SERP", "who ranks for", "rankings for", "search results in", "mobile SERP", "local pack", "people also ask", "ads on Google for", "this keyword in Germany", "desktop vs mobile results". Pass country, language, city, and device when the user names them. A fast organic-only lookup is hasdata-serp-light. Google AI Mode is hasdata-ai-mode. The AI Overview box is hasdata-ai-overview.
allowed-tools:
  - Bash(hasdata *)
---

# Full Google SERP

## How to fetch

1. If a HasData tool is connected whose name contains `google_serp_serp` and does not contain `light`, call it. Read its schema. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector. Do not install software and do not ask for an API key.
3. Use `hasdata google-serp` only when the connector cannot be connected and the binary is already on PATH.

## What to pass

`q` is required. Set the rest from what the user actually said:

| They said | Field |
| --- | --- |
| A country | `gl` (two letters, such as us, de, gb) |
| A language | `hl`, and `lr` when they want pages written in that language |
| A city or "near me" market | `location` (Google's canonical place name) |
| A Google domain such as google.de | `domain` |
| Phone or desktop | `deviceType` |
| Page size | `num` (10 to 100) |
| Page 2 or deeper | `start` (0 is page 1) |
| Past hour, day, week, month, year | `tbs` with `qdr:h`, `qdr:d`, `qdr:w`, `qdr:m`, or `qdr:y` |
| Do not autocorrect this query | `nfpr` set to 1 |

Do not invent `uule`, `ludocid`, `kgmid`, or `lsig`. Pass those only when a previous HasData response returned them.

## What to return

Position, title, link, and snippet for the organic results. Then, only when the response includes them: ads, people also ask, related searches, the knowledge panel, and the local pack. If an `aiOverview` block is present, say so and offer to expand it with [hasdata-ai-overview](../hasdata-ai-overview/SKILL.md) immediately, because that token expires in about a minute.

A cheap organic lookup without ads or the local pack is [hasdata-serp-light](../hasdata-serp-light/SKILL.md).
