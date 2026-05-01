---
name: hasdata-cli-installation
description: |
  Install the official HasData CLI and configure authentication.
  Releases: https://github.com/hasdata/cli/releases
  Docs: https://docs.hasdata.com
  Get an API key: https://app.hasdata.com/api-keys
---

# HasData CLI Installation

## Quick Install

The CLI is a single Go binary. Install with the official one-liner:

```bash
curl -fsSL https://raw.githubusercontent.com/hasdata/cli/main/install.sh | sh
```

This places `hasdata` in `~/.local/bin` (or `/usr/local/bin` on systems where it's writable). Make sure that directory is in `PATH`.

### Manual download

Grab a release binary from https://github.com/hasdata/cli/releases for your platform, `chmod +x`, and move into `PATH`:

```bash
mv hasdata-darwin-arm64 /usr/local/bin/hasdata
chmod +x /usr/local/bin/hasdata
```

### Update

```bash
hasdata update
```

## Authenticate

Get an API key at https://app.hasdata.com/api-keys, then run:

```bash
hasdata configure
```

This prompts for the key and writes `~/.hasdata/config.yaml`. To configure non-interactively:

```bash
hasdata configure --api-key "YOUR-API-KEY" --non-interactive
```

Alternatively, export it as an environment variable (no `configure` step needed):

```bash
export HASDATA_API_KEY="YOUR-API-KEY"
```

Add to `~/.zshrc` / `~/.bashrc` for persistence. The `--api-key` flag on any command overrides both the env var and the config file.

## Verify

Print the version and run one small request:

```bash
hasdata version
mkdir -p .hasdata
hasdata google-serp --q "hello world" --pretty -o .hasdata/install-check.json
```

The install is healthy when both succeed and `install-check.json` contains organic results.

## Troubleshooting

**`command not found: hasdata`** — `~/.local/bin` is not in `PATH`. Add it:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

**`401 Unauthorized`** — API key is missing or invalid. Re-run `hasdata configure` or check `HASDATA_API_KEY`.

**`429 Too Many Requests`** — rate limit hit. The CLI auto-retries (`--retries 2` default). Increase with `--retries 5` or wait.

**Self-hosted / custom endpoint** — set `--endpoint https://your-instance.com` per call, or pass it to `hasdata configure --endpoint ...`.
