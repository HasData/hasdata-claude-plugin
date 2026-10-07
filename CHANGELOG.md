# Changelog

All notable changes to this plugin. Versions follow [semver](https://semver.org/).

## [1.1.4] — 2026-10-07

### Added

- The plugin bundles the HasData connector at `https://mcp.hasdata.com/mcp`, the
  same OAuth server as the directory listing. On claude.ai it shows up as a
  connector to connect. No API key is stored in the plugin.
- Ten skills for sources the connector already serves: YouTube, TikTok, Facebook,
  Google News, Google Images, Google Trends, Google Scholar, Google Hotels and
  Booking.com, Walmart, and a price comparison across Amazon, Walmart, and
  Google Shopping. The Instagram skill no longer says TikTok and Facebook have
  no API.
- Commands for search, news, a YouTube transcript, hotels, reviews, and jobs.
  On claude.ai a command loads as a skill. The price command also checks Walmart.
- An agent for Cowork and Claude Code that runs multi-step data jobs through
  the connector. Chat does not load agents.
- A session-start note, on Cowork and Claude Code, to use the connector instead
  of installing a CLI. Chat does not load hooks.
- Terms of use URL for the directory listing.
- Maps, scraping, Amazon, Shopify, Zillow, Redfin, Airbnb, Indeed, Glassdoor,
  Yelp, YellowPages, and flights now call the connector first. The shell examples
  are a fallback for a binary that is already installed.
- Hotels and Booking.com stay on the hotels skill. The real-estate skill no
  longer answers those questions, and it no longer tells Claude to call the
  HasData API directly.
- Lead lists, price checks, and company enrichment are connector steps. They
  no longer promise a local file, a schedule, or a personal-email harvest.
  A person lookup requires a company. The umbrella table no longer calls
  Google AI Mode an AI Overview.
- Google search is split into the tools people actually call: full results page,
  fast organic results, AI Mode, the AI Overview box, Google Shopping, short
  videos, and events. AI Mode and the AI Overview are different tools. The
  overview needs a token from the full results page, and that token expires in
  about a minute.

### Changed

- The main skill calls those tools first. When they are not connected, it tells
  the user to press Connect instead of installing a binary or pasting a key into
  the chat. The local `hasdata` command remains the fallback only where that
  binary is already installed.
- Directory description is the two-sentence card text. The manifest now also
  carries documentation, support and privacy URLs.

## [1.1.3] — 2026-10-01

### Changed

- The README installs the CLI from the release archive.

## [1.1.2] — 2026-09-28

Directory release of the icon and install-rule fixes recorded under 1.1.1.

## [1.1.1] — 2026-09-28

### Fixed

- The directory scan flagged the plugin for shipping no icon. Added `icon.svg`, a 512×512
  mark, and pointed `plugin.json` at it.
- `skills/hasdata-cli/rules/install.md` told Claude to pipe a remote install script into a
  shell. That file is a rule Claude acts on rather than prose a person reads, so the command
  ran on the user's machine and what it fetched was never part of the reviewed plugin. The
  macOS and Linux section now installs from the release archive. The one-line script stays
  documented in the CLI repository for anyone who wants it.

## [1.1.0] — 2026-08-11

### Fixed

- Install instructions in the README pointed at `/plugin` search, which only resolves for a
  marketplace the user has already added. Replaced with the two commands that work from a
  cold start on every platform: `claude plugin marketplace add HasData/hasdata-claude-plugin`
  followed by `claude plugin install hasdata@hasdata`.
- Windows had no working path to the CLI. The upstream install script exits with an error
  there, and `skills/hasdata-cli/rules/install.md` — the file Claude reads when the binary is
  missing — documented only `curl | sh`, `tar`, `mv` and a `~/.zshrc` edit. Added a Windows
  section with the release archive and a PowerShell `PATH` step, plus `go install` as a
  platform-neutral third option.
- Two links in the README returned 404: `docs.hasdata.com/getting-started` (used in the
  scrape example) and `docs.hasdata.com/api-reference`. Now `/quickstart` and `/cli`.
- Flight and stay examples carried hardcoded dates. Nine had already passed, and running them
  verbatim returned `HTTP 422 afterOrEqual`. Every date literal is now a placeholder with the
  constraint stated, so the examples cannot expire again.

### Added

- `allowed-tools` on all three slash commands. Previously only the ten skills were scoped.
- A scope section in `/hasdata:enrich` stating that it reads public professional sources
  only, and instructing it to decline requests aimed at a private individual.
- README now documents the seven APIs that shipped in the skills but were missing from it:
  four YouTube endpoints, both Booking.com endpoints, and Google Maps posts.
- `author`, `category`, `version`, `repository` and `keywords` on the marketplace entry.

### Changed

- Plugin description shortened to two plain sentences, and it now discloses the enrichment
  command. The previous version was a long marketplace blurb that ended on a comparison with
  other scrapers.

## [1.0.0] — 2026-08-04

### Fixed

- Install instructions pointed at a repository that did not exist.
- Two command examples used flags the CLI does not have.
- The manifest declared one skill out of ten, so adding this repository as a marketplace
  directly loaded only the umbrella skill.

### Added

- `LICENSE` (MIT), which the manifest had always declared but the repository never shipped.
- Coverage for seven previously undocumented APIs: four YouTube, both Booking.com, and
  Google Maps posts.

## [0.1.0] — 2026-05-01

Initial release: ten skills, three slash commands.
