---
name: hasdata-search
description: |
  Real-time Google and Bing search results with structured JSON output. Use this skill whenever the user asks to search the web, find recent news, look up something on Google or Bing, get Google AI Overviews, check Google Trends, find events, or look up images. Triggers on "search Google for", "Bing search", "find news about", "what's trending", "Google AI Overview for", "find events in", "image search". Returns clean structured JSON — organic results, ads, knowledge graph, related searches — without any HTML scraping. Use this instead of WebSearch when the user wants raw search-engine data, time-filtered results, or location-specific SERPs.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata search APIs

Structured JSON results from Google, Bing, Google News, Google AI Mode, Trends, Events, Images, and short videos.

## When to use

- The user wants raw search-engine results (not just an answer)
- They need news from a specific time window (`--tbs qdr:d|w|m|y`)
- They need location- or device-specific SERPs (`--location`, `--gl`, `--device-type`)
- They want Google AI Overviews, Trends data, image results, or events
- They need to fan out from search results into other APIs (e.g., search → scrape each result)

## APIs in this group

| Command                    | Purpose                                            | Cost |
| -------------------------- | -------------------------------------------------- | ---- |
| `google-serp`              | Full Google SERP (organic, ads, KG, related)       | 10   |
| `google-serp-light`        | Lighter / faster Google SERP                       | 5    |
| `google-news`              | Google News results                                | 10   |
| `google-ai-mode`           | Google AI Overview (AI-generated answer)           | 5    |
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

# Google AI Overview
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
