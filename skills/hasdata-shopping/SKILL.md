---
name: hasdata-shopping
description: |
  Google Shopping results, the immersive product card, and the product panel (offers, specs, reviews). Use this skill when the user wants Google Shopping prices, sellers on a Google product, or the shopping results for a keyword in a country. Triggers on "Google Shopping", "shopping results for", "sellers on Google for", "product panel", "immersive product". Amazon is hasdata-ecommerce. Walmart is hasdata-walmart. A comparison across those stores is hasdata-prices.
allowed-tools:
  - Bash(hasdata *)
---

# Google Shopping

## How to fetch

1. If HasData tools are connected, call the one that matches the step below. Read its schema. Do not run a shell command when the tool exists.
2. If no HasData tool is connected, ask the user to press Connect on the HasData connector. Do not install software and do not ask for an API key.
3. Use `hasdata google-shopping` only when the connector cannot be connected and the binary is already on PATH. The immersive and product-panel steps need tokens from a HasData response; do not invent a CLI flag for them.

## Which tool

| Ask | Tool name contains | Pass |
| --- | --- | --- |
| Shopping results for a query | `google_serp_shopping` | `q`. Also `gl`, `hl`, `location`, `deviceType` when the user names them |
| The immersive product popup | `google_serp_immersive_product` | `pageToken` from `immersiveProductPageToken` on a shopping result. Set `moreStores` when they want every seller, not the first few |
| Offers, specs, or reviews on a product | `google_serp_product` | `productId` from a previous result. `searchType` is `offers`, `specs`, or `reviews` |

`start` pages through shopping results. Do not invent `shoprs` or `uule`.

Return store, title, price, and link from the tool output.
