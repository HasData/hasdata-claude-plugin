# Changelog

All notable changes to this plugin. Versions follow [semver](https://semver.org/).

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
