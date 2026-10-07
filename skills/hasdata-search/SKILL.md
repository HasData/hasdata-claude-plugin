---
name: hasdata-search
description: |
  Bing and DuckDuckGo results, and the index of the Google search skills. Use this skill for a Bing or DuckDuckGo query. For Google, switch to the matching skill: hasdata-serp for the full results page, hasdata-serp-light for a fast organic lookup, hasdata-ai-mode for Google AI Mode, hasdata-ai-overview for the AI Overview box, hasdata-news, hasdata-images, hasdata-trends, hasdata-shopping, hasdata-shorts, or hasdata-events. Triggers on "Bing search", "DuckDuckGo". Do not use this skill for a Google rank check or an AI Overview.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata search APIs

Structured JSON results from Google, Bing, Google News, Google AI Mode, Trends, Events, Images, and short videos.

## When to use

Google has its own skills. Load one of those instead of staying here:

- Full results page, rankings, ads, local pack: [hasdata-serp](../hasdata-serp/SKILL.md)
- Fast organic links for enrichment: [hasdata-serp-light](../hasdata-serp-light/SKILL.md)
- Google AI Mode: [hasdata-ai-mode](../hasdata-ai-mode/SKILL.md)
- The AI Overview box: [hasdata-ai-overview](../hasdata-ai-overview/SKILL.md)
- News, images, trends, shopping, short videos, events: the skill with that name

Stay here for Bing (`bing_serp`, field `q`) and DuckDuckGo (`duckduckgo_serp`, field `q`). DuckDuckGo region is `kl` in country-language form, such as us-en.

## APIs in this group

| Command                    | Purpose                                            | Cost |
| -------------------------- | -------------------------------------------------- | ---- |
| `google-serp`              | Full Google SERP (organic, ads, KG, related)       | 10   |
| `google-serp-light`        | Lighter / faster Google SERP                       | 5    |
| `google-news`              | Google News results                                | 10   |
| `google-ai-mode`           | Google AI Mode (not the AI Overview box)           | 5    |
| `google-trends`            | Google Trends interest data                        | 5    |
| `google-events`            | Google Events listings                             | 5    |
| `google-images`            | Google Images results                              | 5    |
| `google-short-videos`      | Google short videos (Shorts/TikToks/Reels)         | 5    |
| `bing-serp`                | Full Bing SERP                                     | 10   |

## Quick start

```bash
# Google SERP, US English, 20 results
hasdata google-serp --q "best wireless earbuds 2026" --num 20 --pretty -o .hasdata/serp-earbuds.json

# News from the past day, sorted by date
hasdata google-news --q "openai" --pretty -o .hasdata/news-openai.json

# Google AI Mode. The AI Overview box is a different tool: hasdata-ai-overview
hasdata google-ai-mode --q "what is retrieval augmented generation" --pretty -o .hasdata/ai-rag.json

# Time-filtered SERP (past week)
hasdata google-serp --q "claude code release" --tbs "qdr:w" --pretty -o .hasdata/serp-cc-week.json

# Location-targeted SERP
hasdata google-serp --q "coffee near me" --location "Brooklyn,New York,United States" --pretty -o .hasdata/serp-coffee-bk.json

# Bing SERP
hasdata bing-serp --q "openai api pricing" --pretty -o .hasdata/bing-openai.json

# Google Trends
hasdata google-trends --q "claude code" --pretty -o .hasdata/trends-cc.json

# Events near a city
hasdata google-events --q "tech meetups" --location "San Francisco,California,United States" --pretty -o .hasdata/events-sf.json

# Image search
hasdata google-images --q "minimalist desk setup" --pretty -o .hasdata/images-desk.json
```

## Common flags (Google SERP family)

| Flag                  | Purpose                                                            |
| --------------------- | ------------------------------------------------------------------ |
| `--q <query>`         | Search query (required)                                            |
| `--num <n>`           | Results per page, 10–100                                           |
| `--start <n>`         | Pagination offset (0 = page 1, 10 = page 2, …)                     |
| `--gl <code>`         | Country code (e.g. `us`, `de`, `jp`)                               |
| `--hl <code>`         | UI language (e.g. `en`, `de`)                                      |
| `--location <text>`   | Canonical location (e.g. `Austin,Texas,United States`)             |
| `--device-type`       | `desktop` (default) / `mobile` / `tablet`                          |
| `--domain <google.X>` | Google domain (`google.com`, `google.co.uk`, etc.)                 |
| `--tbs <filter>`      | Time / sort filters: `qdr:d`, `qdr:w`, `qdr:m`, `qdr:y`, `sbd:1`   |
| `--safe <active\|off>`| Adult content filter                                               |

Run `hasdata google-serp --help` for the full list (all 200+ Google domains, knowledge graph IDs, etc.).

## Tips

- **`google-serp-light`** is half the cost (5 vs 10 credits) when you don't need ads / knowledge graph / related searches — just organic results.
- For news/articles published recently, use `google-news` + `--tbs qdr:d|w` rather than `google-serp` filtered by time.
- `google-ai-mode` gives you Google's AI-generated answer plus citations — great when you want a summary, not a list.
- Combine with `web-scraping` for "search → scrape top result" flows:
  ```bash
  hasdata google-serp --q "react hooks tutorial" --num 5 -o .hasdata/serp.json
  url=$(jq -r '.organicResults[0].link' .hasdata/serp.json)
  hasdata web-scraping --url "$url" --output-format markdown -o .hasdata/top-result.md
  ```
- Use `jq` to slice large result sets — full responses can include 100+ items per call.

## Working with results

```bash
# Top 10 organic results: title + URL
jq -r '.organicResults[0:10] | .[] | "\(.title)\n  \(.link)\n"' .hasdata/serp.json

# News headlines + sources
jq -r '.newsResults[] | "\(.source)  —  \(.title)\n  \(.link)"' .hasdata/news.json

# AI Mode answer text
jq -r '.aiOverview.text' .hasdata/ai-rag.json

# Trends: interest over time
jq '.interestOverTime' .hasdata/trends.json
```

## See also

- [hasdata-scrape](../hasdata-scrape/SKILL.md) — fetch full content from a result URL
- [hasdata-maps](../hasdata-maps/SKILL.md) — for "near me" / business search use Maps instead of SERP
- [hasdata-ecommerce](../hasdata-ecommerce/SKILL.md) — for product searches use Amazon/Shopping APIs instead
