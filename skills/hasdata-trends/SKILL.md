---
name: hasdata-trends
description: |
  Google Trends interest over time and by region. Use this skill when the user wants search interest, a rising or falling term, or a comparison of queries on Google Trends. Triggers on "Google Trends", "search interest for", "is this keyword rising", "compare these terms on Trends". Returns structured JSON from the Trends tool.
allowed-tools:
  - Bash(hasdata *)
---

# Google Trends

## How to fetch

1. If a HasData tool is connected whose name contains `google_trends`, call it. Read its schema. Pass the term in `q`. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector for this chat. Do not install software and do not ask them to paste an API key.
3. Use the `hasdata` command only when the connector cannot be connected and the binary is already on PATH:

```bash
hasdata google-trends --q "TERM" --pretty -o .hasdata/trends.json
```

`geo` is the region (for example US). `date` is the time window the schema allows. `gprop` limits the property (web, youtube, news, images, shopping) when the schema lists it.

Compare terms by following the schema for multiple queries. Report the interest values the tool returned. Do not describe a trend the tool did not return.

News headlines are [hasdata-news](../hasdata-news/SKILL.md). A normal Google results page is [hasdata-search](../hasdata-search/SKILL.md).
