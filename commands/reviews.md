---
description: Reviews for a place, an Amazon product, or a Walmart product
argument-hint: place name, ASIN, or product URL
allowed-tools:
  - Bash(hasdata *)
---

# /hasdata:reviews

Subject: **$ARGUMENTS**

If HasData tools are connected, pick the tool from what the user gave:

- A Google Maps place, restaurant, or local business: the tool whose name contains `google_maps_reviews`. If you only have a name, call `google_maps_search` first and take the place id from that result.
- A Yelp place: the tool whose name contains `yelp_reviews`, after `yelp_search` if you need the place id.
- An Amazon ASIN or amazon.com URL: the tool whose name contains `amazon_reviews`.
- A Walmart item or walmart.com URL: the tool whose name contains `walmart_reviews`.

Read the tool schema. Do not run a shell command when the tool exists.

If no HasData tool is connected, ask the user to press Connect. Do not install software and do not ask for an API key.

Return the rating, the review count, and a few review excerpts that appear in the tool output. Do not invent a review.
