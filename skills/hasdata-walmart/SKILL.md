---
name: hasdata-walmart
description: |
  Walmart product search, product details, and reviews. Use this skill when the user wants a Walmart price, a Walmart item, or reviews on a Walmart product. Triggers on "Walmart search", "price on Walmart", "Walmart reviews", "this Walmart link". Returns structured JSON. For a comparison across Amazon, Walmart, and Google Shopping, use the prices skill.
allowed-tools:
  - Bash(hasdata *)
---

# Walmart

## How to fetch

1. If a HasData tool is connected, call the one whose name matches the task below. Read that tool's schema and fill the fields it asks for. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector for this chat. Do not install software and do not ask them to paste an API key.
3. Use the `hasdata` command only when the connector cannot be connected and the binary is already on PATH. Run `hasdata --help` and use a Walmart command only if it is listed.

## Which tool

| Ask | Tool name contains | Pass |
| --- | --- | --- |
| Search Walmart | `walmart_search` | `q` |
| One product | `walmart_product` | `itemId` or `url` |
| Reviews | `walmart_reviews` | `itemId` or `url` |

`minPrice` and `maxPrice` filter a search. `domain` is the Walmart site when the user names a country. Do not invent a price, rating, or review the tool did not return.

Amazon and Shopify stay on [hasdata-ecommerce](../hasdata-ecommerce/SKILL.md). A cross-store price check is [hasdata-prices](../hasdata-prices/SKILL.md).
