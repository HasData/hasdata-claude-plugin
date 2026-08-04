---
name: hasdata-ecommerce
description: |
  Structured product data from Amazon, Shopify stores, and Google Shopping. Use this skill when the user wants to find products, check prices, get product details, list a Shopify store's catalog, scrape an Amazon seller's products, compare prices across retailers, or pull immersive product data. Triggers on "find products on Amazon", "Amazon search for", "ASIN B0...", "Shopify products from", "this Shopify store", "Google Shopping for", "compare prices for", "what does this seller sell", "Amazon seller", or any e-commerce product / pricing request. Returns structured JSON — title, price, ratings, reviews, images, availability — without HTML scraping.
allowed-tools:
  - Bash(hasdata *)
---

# hasdata e-commerce APIs

Structured catalog and product data from Amazon, Shopify, and Google Shopping.

## When to use

- User wants product info from Amazon (search, single ASIN, seller, seller's products)
- User wants to enumerate or filter a Shopify store's catalog or collections
- User wants Google Shopping cross-retailer prices for a query
- User wants the immersive-product detail card (Google's rich product view)

For non-platform URLs, use [hasdata-scrape](../hasdata-scrape/SKILL.md). For real-estate listings, jobs, or businesses, use the dedicated skill.

## APIs in this group

| Command                       | Purpose                                                          | Cost |
| ----------------------------- | ---------------------------------------------------------------- | ---- |
| `amazon-search`               | Amazon search results for a query                                | 5    |
| `amazon-product`              | Full product page by ASIN (price, ratings, images, sellers)      | 5    |
| `amazon-seller`               | Seller profile by seller ID                                      | 5    |
| `amazon-seller-products`      | All products listed by a seller                                  | 5    |
| `shopify-products`            | Catalog of any Shopify store (by URL)                            | 5    |
| `shopify-collections`         | Collections list for a Shopify store                             | 5    |
| `google-shopping`             | Google Shopping aggregated results (multi-retailer prices)       | 10   |
| `google-immersive-product`    | Google's immersive product detail card                           | 5    |

## Quick start

### Amazon

```bash
# Search Amazon
hasdata amazon-search --q "wireless mouse" --pretty -o .hasdata/amazon-mouse.json

# Sort by price low → high, page 2
hasdata amazon-search --q "yoga mat" --sort-by priceLowToHigh --page 2 --pretty -o .hasdata/amazon-yoga-p2.json

# Single product by ASIN
hasdata amazon-product --asin "B0DHJ7SBDR" --pretty -o .hasdata/amazon-product.json

# Different Amazon domain
hasdata amazon-product --asin "B0DHJ7SBDR" --domain www.amazon.co.uk --pretty -o .hasdata/amazon-uk.json

# Seller's catalog
hasdata amazon-seller --seller-id "ATQQBVXK188KS" --pretty -o .hasdata/seller.json
hasdata amazon-seller-products --seller-id "ATQQBVXK188KS" --pretty -o .hasdata/seller-products.json
```

### Shopify

```bash
# Any Shopify store's products (uses public /products.json)
hasdata shopify-products --url "https://allbirds.com" --limit 250 --pretty -o .hasdata/allbirds.json

# Filter by collection handle
hasdata shopify-products --url "https://allbirds.com" --collection "mens-shoes" --limit 100 --pretty -o .hasdata/allbirds-mens.json

# Store's collection list
hasdata shopify-collections --url "https://allbirds.com" --limit 50 --pretty -o .hasdata/allbirds-cols.json
```

### Google Shopping

```bash
# Cross-retailer search
hasdata google-shopping --q "airpods pro 2" --pretty -o .hasdata/shopping-airpods.json

# Immersive product detail (uses kgmid / product ID from a Google search)
hasdata google-immersive-product --page-token "<immersiveProductPageToken>" --pretty -o .hasdata/immersive.json
```

## Common Amazon flags

| Flag                         | Purpose                                                                |
| ---------------------------- | ---------------------------------------------------------------------- |
| `--q <query>`                | Search query (search only)                                             |
| `--asin <id>`                | Product ASIN (product only)                                            |
| `--seller-id <id>`           | Seller ID (seller / seller-products)                                   |
| `--page <n>`                 | Pagination                                                             |
| `--sort-by`                  | `featured`, `priceLowToHigh`, `priceHighToLow`, `avgCustomerReview`, `newestArrivals`, `bestSellers` |
| `--domain`                   | `www.amazon.com`, `.co.uk`, `.de`, `.co.jp`, `.in`, `.ca`, etc.        |
| `--language`                 | Per-domain language code                                               |
| `--delivery-zip`             | Shipping ZIP for accurate pricing/availability                         |
| `--shipping-location`        | Two-letter country code for delivery                                   |

## Tips

- **Amazon search → product fan-out** is the most common pattern:
  ```bash
  hasdata amazon-search --q "ergonomic chair" --pretty -o .hasdata/chairs.json
  for asin in $(jq -r '.products[].asin' .hasdata/chairs.json | head -10); do
    hasdata amazon-product --asin "$asin" --pretty -o ".hasdata/chair-$asin.json" &
  done
  wait
  ```
- **Shopify** doesn't need API keys per store — `shopify-products` works against any storefront's public `/products.json`. `--limit 250` is the max.
- For **price tracking**, always pass `--delivery-zip` on Amazon — prices and Prime eligibility vary by location.
- **Google Shopping** is best when comparing prices across retailers; **Amazon** alone is faster when you only need Amazon listings.
- `google-immersive-product` takes `--page-token`, the `immersiveProductPageToken` returned by
  `google-shopping` — it's not directly user-facing, always chain it after a Shopping call.

## Working with results

```bash
# Amazon search → top 10 with title, price, rating, ASIN
jq -r '.products[0:10] | .[] | "\(.title)\t$\(.price.value // "n/a")\t\(.rating // "n/a")★\t\(.asin)"' .hasdata/amazon.json

# Amazon product → key fields
jq '{title, price: .price.value, rating, reviewsCount, mainImage}' .hasdata/amazon-product.json

# Shopify products → name, price, vendor
jq -r '.products[] | "\(.title)\t$\(.variants[0].price)\t\(.vendor)"' .hasdata/shopify.json

# Google Shopping → all retailer prices for a product
jq '.shoppingResults[] | {title, source, price, link}' .hasdata/shopping.json
```

## See also

- [hasdata-scrape](../hasdata-scrape/SKILL.md) — for non-Amazon, non-Shopify product pages
- [hasdata-search](../hasdata-search/SKILL.md) — `google-serp` if you want general results not Shopping-specific
