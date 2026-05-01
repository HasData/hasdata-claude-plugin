---
name: hasdata-scrape
description: |
  Scrape any public URL into clean HTML, text, markdown, or AI-extracted JSON. Supports JavaScript-rendered pages (SPAs), CSS-selector extraction, AI-driven schema extraction, screenshots, custom headers, and click/fill page interactions. Use this skill whenever the user gives a URL and wants its contents, says "scrape", "fetch", "grab the page", "extract X from this site", "render this SPA", "screenshot this page", "click through and get content", or wants structured fields from a page. Triggers also on "extract emails from", "get the title and price from", "scroll and click then scrape". Use this instead of WebFetch when the user wants AI-driven structured extraction, JS-rendered content, screenshots, or page interactions before extracting.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata web-scraping

Scrape any URL into HTML, text, markdown, or AI-extracted JSON. Supports JS rendering, screenshots, custom headers, CSS-selector extraction, AI-driven schema extraction, and page interaction scenarios.

## When to use

- User gives a specific URL and wants its content
- The page is JS-rendered (SPA) — `--js-rendering` is on by default
- The site blocks scrapers — escalate to `--proxy-type residential`
- User needs structured data via CSS selectors (`--extract-rules`) or AI (`--ai-extract-rules-json`)
- User needs to interact with the page (click, fill, scroll) before extracting — use `--js-scenario-json`

For a webpage with a dedicated HasData API (Amazon, Zillow, Yelp, Indeed, etc.), prefer the **platform-specific API** instead — it returns structured JSON with no parsing.

## Quick start

```bash
# Markdown (LLM-optimized)
hasdata web-scraping --url "https://example.com" --output-format markdown -o .hasdata/example.md

# Plain text
hasdata web-scraping --url "https://example.com" --output-format text -o .hasdata/example.txt

# Raw HTML
hasdata web-scraping --url "https://example.com" --output-format html -o .hasdata/example.html

# Both markdown and links in JSON
hasdata web-scraping --url "https://example.com" --output-format markdown,links --pretty -o .hasdata/example.json

# Disable JS rendering for static sites (faster)
hasdata web-scraping --url "https://example.com" --no-js-rendering --output-format html -o .hasdata/example.html

# Wait for a CSS selector before extracting (dynamic content)
hasdata web-scraping --url "https://app.example.com/dashboard" --wait-for "#main-content" -o .hasdata/dash.html

# Use residential proxies for hard-to-scrape sites
hasdata web-scraping --url "https://www.target-site.com" --proxy-type residential --proxy-country US -o .hasdata/page.html

# Take a screenshot
hasdata web-scraping --url "https://example.com" --screenshot -o .hasdata/example.json --pretty
```

## CSS-selector extraction (cheap, deterministic)

Pull specific fields by passing CSS selectors via `--extract-rules`:

```bash
hasdata web-scraping --url "https://news.ycombinator.com" \
  --extract-rules '{"titles": "span.titleline > a", "scores": "span.score"}' \
  --pretty -o .hasdata/hn.json
```

Selector suffixes:
- `selector` — text content
- `selector @attr` — attribute value (e.g. `a#link @href`)

For larger rule sets, load from a file: `--extract-rules-json @rules.json` (or `-` for stdin).

## AI-driven extraction (smart, more credits)

When the structure varies or selectors are brittle, use AI extraction. Define output schema as JSON; the API extracts matching fields:

```bash
hasdata web-scraping --url "https://example.com/pricing" \
  --ai-extract-rules-json '{
    "plans": {
      "type": "list",
      "output": {
        "name": {"type": "string"},
        "price": {"type": "number"},
        "features": {"type": "list"}
      }
    }
  }' \
  --pretty -o .hasdata/pricing.json
```

Supported types: `string`, `number`, `boolean`, `list`, `item` (nested object via `output`).

## JS scenarios (interact, then scrape)

Run actions on the page before extracting:

```bash
hasdata web-scraping --url "https://example.com/login" \
  --js-scenario-json '[
    {"fill": ["#email", "user@example.com"]},
    {"fill": ["#password", "secret"]},
    {"click": "button[type=submit]"},
    {"waitFor": "#dashboard"}
  ]' \
  --output-format markdown -o .hasdata/dashboard.md
```

Action types: `evaluate`, `click`, `wait`, `waitFor`, `waitForAndClick`, `scrollX`, `scrollY`, `fill`. Actions execute sequentially.

## Common flags

| Flag                          | Purpose                                                                       |
| ----------------------------- | ----------------------------------------------------------------------------- |
| `--url <url>`                 | Target URL (required)                                                         |
| `--output-format <fmt>`       | `html`, `text`, `markdown`, `json`, or comma-list (returns JSON)              |
| `--js-rendering` (default on) | Render JS — disable with `--no-js-rendering` for static sites                 |
| `--wait <ms>`                 | Wait N ms after load                                                          |
| `--wait-for <selector>`       | Wait for a CSS selector to appear                                             |
| `--proxy-type`                | `datacenter` (default) / `residential`                                        |
| `--proxy-country`             | `US`, `UK`, `DE`, `IE`, `FR`, `IT`, `SE`, `BR`, `CA`, `JP`, `SG`, `IN`, `ID`  |
| `--block-ads` (default on)    | Block ads / disable with `--no-block-ads`                                     |
| `--block-resources` (default) | Block images/CSS — disable with `--no-block-resources`                        |
| `--block-urls <urls>`         | Specific URLs to block during load                                            |
| `--include-only-tags <css>`   | Only include matching elements                                                |
| `--exclude-tags <css>`        | Exclude matching elements                                                     |
| `--extract-emails` (default)  | Pull emails from the page                                                     |
| `--extract-links`             | Pull all links                                                                |
| `--extract-rules <json>`      | CSS-selector extraction rules                                                 |
| `--ai-extract-rules-json`     | AI-driven extraction schema                                                   |
| `--js-scenario-json`          | Page interaction scenario                                                     |
| `--screenshot` (default on)   | Capture a screenshot — disable with `--no-screenshot`                         |
| `--headers-json <json>`       | Custom request headers                                                        |
| `--remove-base64-images`      | Strip base64-embedded images from output                                      |

## Tips

- **`--output-format markdown`** is the cheapest, easiest format for LLM consumption. Use it unless you specifically need HTML or structured JSON.
- **Static sites** are 2–3× faster with `--no-js-rendering`. Default is on because most modern sites need it.
- **Single format outputs raw content** (e.g. `markdown` returns markdown text). **Multiple formats output JSON** with keys per format.
- If a scrape returns near-empty content with default settings, escalate with `--proxy-type residential --proxy-country US --no-block-resources` to retrieve the full page.
- Combine `--extract-rules` with `--output-format markdown` to get both the cleaned page and named fields in one call.
- Always quote the URL — `?` and `&` are shell-special.

## Working with results

```bash
# Markdown output → grep / head directly
head -50 .hasdata/example.md
grep -i "pricing" .hasdata/example.md

# JSON (multi-format) output
jq '.markdown' .hasdata/example.json
jq '.links[]' .hasdata/example.json

# AI extraction result
jq '.aiExtract.plans[]' .hasdata/pricing.json

# Screenshot is base64 in JSON — decode to file
jq -r '.screenshot' .hasdata/page.json | base64 -d > .hasdata/page.png
```

## See also

- [hasdata-search](../hasdata-search/SKILL.md) — find URLs first, then scrape them
- [hasdata-ecommerce](../hasdata-ecommerce/SKILL.md) — Amazon/Shopify URLs have dedicated APIs
- [hasdata-realestate](../hasdata-realestate/SKILL.md) — Zillow/Redfin/Airbnb URLs have dedicated APIs
- [hasdata-maps](../hasdata-maps/SKILL.md) — Google Maps URLs have dedicated APIs
