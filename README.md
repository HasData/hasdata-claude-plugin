# HasData Plugin for Claude Code

Real-time web data inside Claude Code. Pull structured JSON from Google (SERP, Maps, News, Trends, Flights, Images, Events), Bing, Amazon, Shopify, Zillow, Redfin, Airbnb, Yelp, YellowPages, Indeed, Glassdoor, and Instagram — or scrape any URL into clean HTML, markdown, or AI-extracted fields.

This plugin adds the [HasData CLI](https://github.com/HasData/hasdata-cli) as a set of skills, so Claude Code can answer questions and produce data that depends on the live web.

## Features

- **Search** — Google SERP, News, AI Overviews, Trends, Events, Images, short videos; Bing SERP
- **Scrape** — Any URL into HTML, text, markdown, or AI-extracted structured JSON, with screenshot capture and CSS / AI extraction rules
- **Maps** — Google Maps place search, full place profiles, reviews, photos, posts
- **E-commerce** — Amazon (search, product, seller), Shopify (catalog, collections), Google Shopping
- **Real estate** — Zillow / Redfin listings + property details, Airbnb search + listings
- **Jobs** — Indeed and Glassdoor search + single-listing details
- **Business directories** — Yelp and YellowPages search + place profiles
- **Social** — Instagram public profiles; YouTube search, video and channel data, full transcripts
- **Travel** — Google Flights search with rich filters, Booking.com search + property details

Dedicated APIs return ready-to-use structured JSON, with no HTML parsing and no selectors to maintain.

## Installation

### 1. Install the plugin

Two commands, same on macOS, Linux and Windows:

```bash
claude plugin marketplace add HasData/hasdata-claude-plugin
claude plugin install hasdata@hasdata
```

If Claude Code is already running, `/reload-plugins` picks it up without a restart.

The plugin is also published in Anthropic's community marketplace, which may lag this
repository by a day:

```bash
claude plugin marketplace add anthropics/claude-plugins-community
claude plugin install hasdata@claude-community
```

To work on the plugin itself, load it from a checkout instead:

```bash
claude --plugin-dir ./hasdata-claude-plugin
```

### 2. Install the HasData CLI

The plugin shells out to the `hasdata` binary.

**macOS / Linux** — the official one-liner (verifies SHA-256 checksums, detects OS and arch):

```bash
curl -fsSL https://raw.githubusercontent.com/HasData/hasdata-cli/main/install.sh | sh
```

**With Go:**

```bash
go install github.com/HasData/hasdata-cli@latest
```

**Windows** — the install script does not run here. Download the `.zip` for your architecture
from the [releases page](https://github.com/HasData/hasdata-cli/releases), extract it, and add
`hasdata.exe` to `%PATH%`.

### 3. Authenticate

Get an API key at <https://app.hasdata.com/api-keys>, then run:

```bash
hasdata configure
```

This stores the key in `~/.hasdata/config.yaml`.

You can also export it as an environment variable (add to `~/.zshrc` / `~/.bashrc` for persistence):

```bash
export HASDATA_API_KEY="YOUR-API-KEY"
```

### 4. Verify

```bash
hasdata version
hasdata google-serp --q "hello world" --pretty
```

You should see structured search results.

## Usage

Once installed, Claude Code uses HasData automatically when web data is needed. Just ask naturally:

**Search the web:**
```
Search Google for "best practices for React testing" and summarize the recommendations
```

**Scrape a page:**
```
Scrape https://docs.hasdata.com/quickstart and extract the auth steps
```

**Find local businesses:**
```
Get the top 10 pizza places in Brooklyn from Google Maps with phone numbers
```

**Compare products:**
```
Find wireless mice under $50 on Amazon, sorted by rating
```

**Real estate:**
```
List for-sale homes in Austin TX between $500k and $800k, 3+ beds
```

**Jobs:**
```
Find remote Rust engineering jobs on Indeed, then summarize the top 5
```

**Flights:**
```
Cheapest non-stop round trip JFK → LHR for July 10–20
```

**Social:**
```
Pull the Instagram profile for @hasdatadotcom
```

### Commands used under the hood

| Category     | CLI commands                                                                |
| ------------ | --------------------------------------------------------------------------- |
| Search       | `google-serp`, `google-serp-light`, `google-news`, `google-ai-mode`, `bing-serp`, `google-trends`, `google-events`, `google-images`, `google-short-videos` |
| Scrape       | `web-scraping`                                                              |
| Maps         | `google-maps`, `google-maps-place`, `google-maps-reviews`, `google-maps-contributor-reviews`, `google-maps-photos`, `google-maps-posts` |
| E-commerce   | `amazon-search`, `amazon-product`, `amazon-seller`, `amazon-seller-products`, `shopify-products`, `shopify-collections`, `google-shopping`, `google-immersive-product` |
| Real estate  | `zillow-listing`, `zillow-property`, `redfin-listing`, `redfin-property`, `airbnb-listing`, `airbnb-property` |
| Jobs         | `indeed-listing`, `indeed-job`, `glassdoor-listing`, `glassdoor-job`        |
| Business     | `yelp-search`, `yelp-place`, `yellowpages-search`, `yellowpages-place`      |
| Social       | `instagram-profile`, `youtube-search-api`, `youtube-video-api`, `youtube-channel-api`, `youtube-transcript-api` |
| Travel       | `google-flights`, `booking-search`, `booking-place`                         |

Run `hasdata --help` for the full list, or `hasdata <command> --help` for per-API flags.

### Output convention

Results are written to a `.hasdata/` directory in your project to keep the context window clean:

```
.hasdata/serp-react_hooks.json
.hasdata/amazon-wireless_mouse.json
.hasdata/maps-pizza-brooklyn.json
.hasdata/zillow-austin-forSale.json
.hasdata/scrape-example.com.md
```

Add `.hasdata/` to your `.gitignore`:

```bash
echo '.hasdata/' >> .gitignore
```

## Configuration

| Variable           | Required                                | Description                                |
| ------------------ | --------------------------------------- | ------------------------------------------ |
| `HASDATA_API_KEY`  | Yes (if not using `hasdata configure`)  | Your HasData API key                       |

You can also pass `--api-key`, `--timeout`, `--retries`, `--pretty`, `-o <path>` to any command.

## Slash commands

Type any of these in Claude Code after the plugin is loaded:

| Command                                   | What it does                                                                              |
| ----------------------------------------- | ----------------------------------------------------------------------------------------- |
| `/hasdata:leads <category> in <city>`     | Local lead list (Google Maps + YellowPages, paginated, deduped TSV)                       |
| `/hasdata:price <product>`                | Cross-retailer price compare (Amazon + Google Shopping); appends to a price-history log + offers `/schedule` for monitoring |
| `/hasdata:enrich <company, or contact + company>` | B2B enrichment from public sources — official site, LinkedIn, GitHub, X, Crunchbase. Business context only; it declines anything aimed at a private individual |

## Skills

The plugin ships one umbrella skill (`hasdata`, always loaded) that routes to specialized skills loaded on demand:

- `hasdata-search` — SERPs, news, AI Overviews, trends, events, images
- `hasdata-scrape` — Web Scraping API
- `hasdata-maps` — Google Maps places, reviews, photos
- `hasdata-ecommerce` — Amazon, Shopify, Google Shopping
- `hasdata-realestate` — Zillow, Redfin, Airbnb
- `hasdata-jobs` — Indeed, Glassdoor
- `hasdata-business` — Yelp, YellowPages
- `hasdata-social` — Instagram
- `hasdata-flights` — Google Flights

## Resources

- [HasData Documentation](https://docs.hasdata.com)
- [HasData CLI Repository](https://github.com/HasData/hasdata-cli)
- [CLI documentation](https://docs.hasdata.com/cli)
- [Get an API key](https://app.hasdata.com/api-keys)

## License

MIT
