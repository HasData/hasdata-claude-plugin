# HasData

Real-time structured data in Claude: Google results, AI Mode, Maps, Amazon, Walmart, Zillow, Indeed, Yelp, YouTube, TikTok, flights, and any public URL as markdown.

On claude.ai and in Cowork, connect the HasData connector. That is OAuth. Do not install a CLI and do not paste an API key into the chat.

The [HasData CLI](https://github.com/HasData/hasdata-cli) is an optional fallback for Claude Code, and only when that binary is already installed.

## Features

- **Search** — full Google results, fast organic results, Google AI Mode, and the AI Overview box as a separate step. Also News, Trends, Images, short videos, Shopping, Bing, and DuckDuckGo
- **Scrape** — Any URL into HTML, text, markdown, or AI-extracted structured JSON, with screenshot capture and CSS / AI extraction rules
- **Maps** — Google Maps place search, full place profiles, reviews, photos, posts
- **E-commerce** — Amazon (search, product, seller), Shopify (catalog, collections), Google Shopping
- **Real estate** — Zillow / Redfin listings + property details, Airbnb search + listings
- **Jobs** — Indeed and Glassdoor search + single-listing details
- **Business directories** — Yelp and YellowPages search + place profiles
- **Social** — Instagram profiles and posts, YouTube including transcripts, TikTok, and a public Facebook profile
- **Travel** — Google Flights. Hotels are Google Hotels and Booking.com, separate from home listings

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

### 2. Optional: the HasData CLI, Claude Code only

Skip this on claude.ai and in Cowork. The plugin does not require the binary. Use it only when the HasData connector cannot be connected and you already want a local `hasdata` command.

**macOS / Linux.** Download the archive for your platform from the
[releases page](https://github.com/HasData/hasdata-cli/releases), then unpack it and put the
binary on `PATH`:

```bash
tar -xzf hasdata_*.tar.gz
mv hasdata /usr/local/bin/hasdata
chmod +x /usr/local/bin/hasdata
```

Use `~/.local/bin` instead when `/usr/local/bin` is not writable, and make sure the directory
is in `PATH`.

The CLI repository also documents a one-line install script that verifies SHA-256 checksums and
detects OS and architecture. Use it from there if you prefer it.

**With Go:**

```bash
go install github.com/HasData/hasdata-cli@latest
```

**Windows.** The install script does not run here. Download the `.zip` for your architecture
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
| `/hasdata:price <product>`                | Cross-retailer price compare (Amazon, Walmart, Google Shopping)                            |
| `/hasdata:enrich <company, or contact + company>` | B2B enrichment from public sources — official site, LinkedIn, GitHub, X, Crunchbase. Business context only; it declines anything aimed at a private individual |
| `/hasdata:search <query>`                 | Fast Google organic results                                                                 |
| `/hasdata:serp <query>`                   | Full Google results page for a rank check                                                   |
| `/hasdata:ai <query>`                     | Google AI Mode answer                                                                       |
| `/hasdata:news <topic>`                   | Google News headlines                                                                      |
| `/hasdata:transcript <YouTube URL or id>` | YouTube transcript, then a short summary                                                   |
| `/hasdata:hotels <city and dates>`        | Hotel prices from Google Hotels and Booking.com                                            |
| `/hasdata:reviews <place or product>`     | Reviews from Maps, Yelp, Amazon, or Walmart                                                |
| `/hasdata:jobs <role and city>`           | Indeed and Glassdoor listings                                                              |

## Skills

The plugin ships one umbrella skill (`hasdata`, always loaded) that routes to specialized skills loaded on demand:

- `hasdata-serp` — full Google results page: rankings, ads, local pack
- `hasdata-serp-light` — fast Google organic results
- `hasdata-ai-mode` — Google AI Mode
- `hasdata-ai-overview` — the AI Overview box on a results page
- `hasdata-shopping` — Google Shopping, immersive product, product panel
- `hasdata-shorts` — Google short videos
- `hasdata-events` — events from Google results
- `hasdata-search` — Bing and DuckDuckGo
- `hasdata-news` — Google News
- `hasdata-images` — Google Images
- `hasdata-trends` — Google Trends
- `hasdata-scholar` — Google Scholar
- `hasdata-scrape` — Web Scraping API
- `hasdata-maps` — Google Maps places, reviews, photos
- `hasdata-ecommerce` — Amazon, Shopify, Google Shopping
- `hasdata-walmart` — Walmart search, product, reviews
- `hasdata-prices` — price comparison across Amazon, Walmart, and Google Shopping
- `hasdata-realestate` — Zillow, Redfin, Airbnb
- `hasdata-hotels` — Google Hotels and Booking.com
- `hasdata-jobs` — Indeed, Glassdoor
- `hasdata-business` — Yelp, YellowPages
- `hasdata-social` — Instagram
- `hasdata-youtube` — YouTube search, video, channel, transcript
- `hasdata-tiktok` — TikTok profile, posts, search, comments
- `hasdata-facebook` — Facebook public profile
- `hasdata-flights` — Google Flights

## Resources

- [HasData Documentation](https://docs.hasdata.com)
- [HasData CLI Repository](https://github.com/HasData/hasdata-cli)
- [CLI documentation](https://docs.hasdata.com/cli)
- [Get an API key](https://app.hasdata.com/api-keys)

## License

MIT
