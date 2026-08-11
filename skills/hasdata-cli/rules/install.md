---
name: hasdata-cli-installation
description: |
  Install the official HasData CLI and configure authentication.
  Releases: https://github.com/HasData/hasdata-cli/releases
  Docs: https://docs.hasdata.com
  Get an API key: https://app.hasdata.com/api-keys
---

# HasData CLI Installation

## Install

The CLI is a single Go binary. Pick the section matching the user's platform — check it
before suggesting a command, because the one-liner below deliberately refuses to run on
Windows.

### macOS and Linux

```bash
curl -fsSL https://raw.githubusercontent.com/HasData/hasdata-cli/main/install.sh | sh
```

This places `hasdata` in `/usr/local/bin`, falling back to `~/.local/bin` when that is not
writable. Make sure the directory is in `PATH`.

Manual alternative — download the archive for the platform from
https://github.com/HasData/hasdata-cli/releases, then:

```bash
tar -xzf hasdata_*.tar.gz
mv hasdata /usr/local/bin/hasdata
chmod +x /usr/local/bin/hasdata
```

### Windows

The install script exits with an error on Windows; do not suggest it there. Download the
`.zip` for the architecture — `hasdata_<version>_Windows_x86_64.zip` or `..._arm64.zip` —
from https://github.com/HasData/hasdata-cli/releases, extract it, and put `hasdata.exe`
on `PATH`. In PowerShell:

```powershell
Expand-Archive -Path .\hasdata_*_Windows_x86_64.zip -DestinationPath "$env:LOCALAPPDATA\hasdata" -Force
[Environment]::SetEnvironmentVariable("PATH", "$env:PATH;$env:LOCALAPPDATA\hasdata", "User")
```

Open a new terminal afterwards so the updated `PATH` takes effect.

### Any platform, with Go installed

```bash
go install github.com/HasData/hasdata-cli@latest
```

Puts the binary in `$(go env GOPATH)/bin`, which must be on `PATH`.

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

Add to `~/.zshrc` / `~/.bashrc` for persistence. On Windows, the equivalent is:

```powershell
[Environment]::SetEnvironmentVariable("HASDATA_API_KEY", "YOUR-API-KEY", "User")
```

The `--api-key` flag on any command overrides both the env var and the config file.

## Verify

Print the version and run one small request:

```bash
hasdata version
mkdir -p .hasdata
hasdata google-serp --q "hello world" --pretty -o .hasdata/install-check.json
```

The install is healthy when both succeed and `install-check.json` contains organic results.

## Troubleshooting

**`command not found: hasdata`** — the install directory is not in `PATH`.

macOS / Linux:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

Windows — confirm where it landed and whether the shell can see it:

```powershell
Get-Command hasdata
```

If that returns nothing, re-add the extract directory to `PATH` as shown above and open a
new terminal.

**`401 Unauthorized`** — API key is missing or invalid. Re-run `hasdata configure` or check `HASDATA_API_KEY`.

**`429 Too Many Requests`** — rate limit hit. The CLI auto-retries (`--retries 2` default). Increase with `--retries 5` or wait.

**Self-hosted / custom endpoint** — set `--endpoint https://your-instance.com` per call, or pass it to `hasdata configure --endpoint ...`.
