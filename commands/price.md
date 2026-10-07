---
description: Compare live prices for one product on Amazon, Walmart, and Google Shopping
argument-hint: product name
allowed-tools:
  - Bash(hasdata *)
---

# /hasdata:price

Product: **$ARGUMENTS**

## Connector

If HasData tools are connected, call all three and stop. Do not run a shell command. Do not promise a price history file or a schedule.

| Store | Tool name contains | Pass |
| --- | --- | --- |
| Amazon | `amazon_search` | the product name |
| Walmart | `walmart_search` | `q` |
| Google Shopping | `google_serp_shopping` | `q` |

Reply with a table: store, title, price, link. Name the cheapest row among prices the tools actually returned. If a store returned nothing, say so.

If no HasData tool is connected, ask the user to press Connect. In Claude Code, authenticate with `/mcp`. Do not install a binary and do not ask for an API key.

## CLI fallback

Only when the connector cannot be connected and `hasdata` is already on PATH: `hasdata amazon-search`, and `hasdata google-shopping`. A Walmart CLI command only if `hasdata --help` lists one. Do not install the CLI to reach this section.
