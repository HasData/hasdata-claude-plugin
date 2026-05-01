---
description: Compare prices for a product across Amazon and Google Shopping, with optional schedule for monitoring over time
argument-hint: <product name>
---

# /hasdata:price

Product: **$ARGUMENTS**

## Steps

1. **Search Amazon** sorted by rating, top 10:

   ```bash
   mkdir -p .hasdata/price-history
   stamp=$(date +%Y%m%d-%H%M%S)
   slug=$(echo "$ARGUMENTS" | tr '[:upper:]' '[:lower:]' | sed -E 's|[^a-z0-9]+|-|g; s|^-||; s|-$||' | cut -c1-60)

   hasdata amazon-search --q "$ARGUMENTS" --sort-by avgCustomerReview \
     --pretty -o ".hasdata/price-history/$slug-amazon-$stamp.json"
   ```

2. **Search Google Shopping** for cross-retailer prices:

   ```bash
   hasdata google-shopping --q "$ARGUMENTS" \
     --pretty -o ".hasdata/price-history/$slug-shopping-$stamp.json"
   ```

3. **Extract a comparable list** from both sources with `jq`:

   ```bash
   echo "=== Amazon (top 10 by rating) ==="
   jq -r '.products[0:10] | .[] | "\(.title)\t$\(.price.value // "n/a")\t\(.rating // "n/a")★\t\(.reviewsCount // 0) reviews\t\(.asin)"' \
     ".hasdata/price-history/$slug-amazon-$stamp.json"

   echo
   echo "=== Google Shopping (cross-retailer) ==="
   jq -r '.shoppingResults[0:15] | .[] | "\(.title)\t$\(.price)\t\(.source)"' \
     ".hasdata/price-history/$slug-shopping-$stamp.json"
   ```

4. **Append a snapshot row** to a per-product price log so trends are visible across runs:

   ```bash
   {
     amazon_min=$(jq '[.products[].price.value | numbers] | min // empty' ".hasdata/price-history/$slug-amazon-$stamp.json")
     amazon_med=$(jq '[.products[].price.value | numbers] | sort | .[length/2|floor] // empty' ".hasdata/price-history/$slug-amazon-$stamp.json")
     shop_min=$(jq '[.shoppingResults[].price | tonumber? ] | min // empty' ".hasdata/price-history/$slug-shopping-$stamp.json" 2>/dev/null)
     shop_med=$(jq '[.shoppingResults[].price | tonumber? ] | sort | .[length/2|floor] // empty' ".hasdata/price-history/$slug-shopping-$stamp.json" 2>/dev/null)
     printf "%s\t%s\t%s\t%s\t%s\n" "$stamp" "$amazon_min" "$amazon_med" "$shop_min" "$shop_med"
   } >> ".hasdata/price-history/$slug.tsv"

   echo
   echo "=== History for '$ARGUMENTS' ==="
   echo "timestamp	amazon_min	amazon_med	shopping_min	shopping_med"
   tail -10 ".hasdata/price-history/$slug.tsv"
   ```

5. **Present a comparison table** for *this* run:
   - **Cheapest overall** (across both sources) with retailer
   - **Best-rated on Amazon** (top rating with ≥100 reviews)
   - **Best value** (low price + high rating)
   - **Price spread** — min / median / max across all retailers
   - If `.hasdata/price-history/$slug.tsv` has 2+ rows, show the **delta vs previous snapshot** (Δ price for amazon_min and shopping_min)

6. **Offer to schedule monitoring.** End the response with:

   > Want me to `/schedule` `/hasdata:price $ARGUMENTS` to run on a recurring cadence (daily / weekly) so the price-history log keeps growing? Routine results land in `.hasdata/price-history/$slug.tsv` for trend analysis.

   If the user says yes, suggest a cadence (`daily` for fast-moving categories like electronics, `weekly` for furniture / appliances) and invoke `/schedule`.

## Cost per run

5 (Amazon) + 10 (Shopping) = 15 credits.
