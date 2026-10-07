---
name: hasdata-prices
description: |
  Compare live prices for one product across Amazon, Walmart, and Google Shopping. Use this skill when the user wants the cheapest offer, a price check, or prices at more than one store. Triggers on "compare prices", "cheapest", "price check for", "how much does this cost", "who sells". Returns a short comparison built only from tool results: store, title, price, and link. Do not invent a price.
allowed-tools:
  - Bash(hasdata *)
---

# Price comparison

## How to fetch

1. If HasData tools are connected, call the shopping tools below. Read each schema and fill the fields it asks for. Do not run a shell command when the tools exist. Do not install software.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector for this chat. Do not ask them to paste an API key.
3. Use the `hasdata` command only when the connector cannot be connected and the binary is already on PATH. Amazon and Google Shopping commands are `amazon-search` and `google-shopping`. A Walmart command is usable only if `hasdata --help` lists it.

## Which tools

Call these for the same product query. Skip a source only if its tool is not connected.

| Store | Tool name contains | Pass |
| --- | --- | --- |
| Amazon | `amazon_search` | the product query |
| Walmart | `walmart_search` | `q` |
| Google Shopping | `google_serp_shopping` | `q` |

If one result needs the full product card, call the tool whose name contains `immersive_product` or `google_serp_product`, using the id or URL from the search result.

## What to show

A short table: store, title, price, availability if the tool returned it, and link. Use only numbers present in the tool output. If a store returned nothing, say that store returned nothing. Name the cheapest row among the prices that actually came back.

A single Amazon, Shopify, or Walmart lookup without a comparison stays on [hasdata-ecommerce](../hasdata-ecommerce/SKILL.md) or [hasdata-walmart](../hasdata-walmart/SKILL.md).
