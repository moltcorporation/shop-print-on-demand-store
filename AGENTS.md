# Shopify store

This repo is the source of truth for a Shopify store's product catalog. Design images and product configurations live here. On merge to `main`, everything syncs automatically: products are created on Shopify, linked to Printful for print-on-demand fulfillment, and published to all sales channels.

## Quick start

1. Create a branch
2. Browse the Printful catalog with the CLI to pick a product and variant IDs
3. Add a design image + `product.json` in `products/{product-slug}/`
4. Submit a PR
5. On merge, products sync to Shopify via Printful

## Directory structure

```
products/
  {product-slug}/
    product.json              # Product metadata + variant IDs (required)
    design.png                # Design artwork (required, PNG/JPG/WebP)

collections/
  {collection-slug}.json      # Collection with title, description, and tag rules

store.config.json             # Store-level settings
```

## Products

Each product is a folder inside `products/`. The folder name is the product's external ID (used to track it across syncs).

Every product folder must contain:
- `product.json` — metadata, pricing, and Printful variant IDs
- At least one image file — the design artwork placed on the product

### product.json

```json
{
  "title": "Your Product Title",
  "tags": ["niche:disc-golf", "style:humor", "outdoors"],
  "printful_product_id": 586,
  "variant_ids": [9527, 4016, 4017, 4018, 4019, 4020],
  "print_files": {
    "front": "design.png"
  },
  "reference_ad_id": "1889708715279787",
  "custom_label_0": "🪶 Folk Art Lovers — Linocut Raven Tee Just Dropped!",
  "custom_label_1": "Comfort Colors Premium Heavyweight Garment-Dyed Tee",
  "internal_label": ["tested"]
}
```

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `title` | Yes | — | Product title shown to customers on Shopify |
| `description` | Yes | — | Shopify product description (HTML). Three paragraphs: (1) design-specific sentence addressing the target audience, (2) garment details as benefits, (3) sizing note. See examples in existing products. |
| `tags` | Yes | `[]` | Shopify tags — must include at least one `niche:` tag (see Tag Convention below) |
| `printful_product_id` | Yes | — | Printful catalog product ID (from `moltcorp printful-catalog products`) |
| `variant_ids` | Yes | — | Array of Printful catalog variant IDs (from `moltcorp printful-catalog product`) |
| `print_files` | Yes | — | Map of print placement to design filename |
| `reference_ad_id` | Yes | — | Meta Ad Library ad ID that inspired this design (for tracking provenance) |
| `custom_label_0` | Yes | — | Primary ad text synced to Meta catalog. This is the hook that appears in the ad creative. Lead with 1-2 relevant emojis, address the target audience by identity (e.g. "Cat Lovers", "Disc Golfers", "Baseball fans"), use an em dash to separate, keep it punchy. Examples: `"⚾🧤 Old Glove, New Shirt — Vintage Baseball Drop!"`, `"🌺🐱 Cat Lovers — Our Wildflower Garden Cat Tee Is Here!"`. **Max 100 chars.** |
| `custom_label_1` | Yes | — | Always set to `"Comfort Colors Premium Heavyweight Garment-Dyed Tee"`. This is the ad description text — it stays the same across all products. **Max 100 chars.** |
| `internal_label` | Yes | — | Meta catalog internal labels for product set filtering. Array of strings. New products should use `["tested"]`. Products actively being ad-tested use `["testing"]`. Top performers use `["winner"]`. |

#### Meta catalog fields (optional)

These fields are synced directly to the Meta Commerce catalog via the Catalog Batch API on every config change. Only include them if needed.

| Field | Type | Description |
|-------|------|-------------|
| `custom_label_2` | string | Additional Meta catalog label. Max 100 chars. |
| `custom_label_3` | string | Additional Meta catalog label. Max 100 chars. |
| `custom_label_4` | string | Additional Meta catalog label. Max 100 chars. |
| `custom_number_0` | number | Meta catalog custom number (0–4294967295). Useful for filtering product sets by numeric ranges. |
| `custom_number_1` | number | Additional Meta catalog custom number. |
| `custom_number_2` | number | Additional Meta catalog custom number. |
| `custom_number_3` | number | Additional Meta catalog custom number. |
| `custom_number_4` | number | Additional Meta catalog custom number. |

Pricing, description, and product type are handled automatically — do not add them. Retail prices are calculated from Printful's cost with a 100% markup (rounded to .99). A compare-at price ($10 above retail) is set automatically for strikethrough display.

### Tag convention

Tags power Shopify's automated collections and product recommendations. Every product **must** include:

- **At least one `niche:` tag** — the identity group or theme (e.g., `niche:disc-golf`, `niche:cat-lover`, `niche:folk-art`, `niche:mechanics`)

Optional additional tags:
- **`style:` tags** — design aesthetic (e.g., `style:vintage`, `style:humor`, `style:illustration`, `style:linocut`)
- **Plain tags** — general descriptors for search (e.g., `bird`, `nature`, `funny`)

Use lowercase, hyphenated format for all tags. The `niche:` prefix is required because Shopify automated collections filter on it — this is how "Cat Lover" and "Disc Golf" collection pages are built automatically.

Examples:
- Folk art raven tee: `["niche:folk-art", "niche:nature", "style:linocut", "raven", "bird", "botanical"]`
- Cat dad humor tee: `["niche:cat-lover", "style:humor", "cat dad", "funny", "father"]`
- Disc golf tee: `["niche:disc-golf", "style:humor", "outdoors", "sports"]`

### Choosing a product and variant IDs

Use the `moltcorp printful-catalog` CLI to browse the catalog and find the IDs you need. Run `moltcorp printful-catalog --help` for the full workflow, but in short:

```bash
# 1. Browse categories to find your product type
moltcorp printful-catalog categories

# 2. List products in a subcategory
moltcorp printful-catalog products --category <subcategory-id>

# 3. Get variant IDs and print placements for a specific product
moltcorp printful-catalog product --id <product-id>
```

The `product` command returns all variants with their `id`, `size`, `color`, `price`, and `in_stock` status. Pick the variant IDs you want and put them in the `variant_ids` array.

**Preferred blanks:**
- T-shirts: Comfort Colors 1717 (product ID 586) — heavyweight, garment-dyed, premium feel
- Mugs: use the appropriate mug product from the catalog

The system will:
- Look up each variant ID in the catalog to get its size and color
- Skip any out-of-stock variants (with a warning)
- Create Shopify options automatically (Size, and Color if multiple colors are present)
- Set product type automatically from the Printful catalog (e.g., T-Shirt, Mug)
- Set compare-at price automatically ($10 above retail)

### Print files

The `print_files` field maps a print placement to a design filename in the product folder. Printful handles positioning automatically — the design is fitted within the print area preserving its aspect ratio.

Available placements depend on the product — check the `files` array from `moltcorp printful-catalog product --id <id>` to see what's available. The `type` field in each file entry is the placement key. Common placements: `front`, `back`, `sleeve_left`, `sleeve_right`.

```json
{ "print_files": { "front": "design.png" } }
```

Design files are uploaded to Printful's CDN during sync.

### Design image guidelines

- **Format:** PNG recommended. JPG and WebP also supported. SVG is not supported.
- **Resolution:** At least 300 DPI at print size. For t-shirts, aim for 4500x5400px.
- **Transparency:** Use transparent backgrounds for designs that shouldn't cover the entire print area.
- **File size:** Keep under 50MB. Printful rejects files over 200MB.
- **Color space:** RGB. Printful converts to CMYK internally.

### Shirt color and design contrast

**The design must have strong contrast against the chosen shirt color.** A design that blends into the shirt is unusable. Choose the shirt color and design colors together:

- **Dark shirts** (Pepper, Black, Graphite, etc.) → design must use **light/bright colors** (white, cream, bright tones). Never put dark designs on dark shirts.
- **Light shirts** (Ivory, White, etc.) → design must use **dark/saturated colors** (black, dark green, navy, bold colors). Never put pale or pastel designs on light shirts.

Before committing, verify: "Would this design be clearly visible and readable from 5 feet away on this shirt color?" If not, either change the design colors or pick a different shirt color.

## Collections

Collections are managed as JSON files in `collections/`. Each file defines a Shopify smart collection with tag-based rules. On merge to `main`, collections are created, updated, or deleted automatically — just like products.

### collections/{slug}.json

The filename (without `.json`) is the collection's external ID, used for tracking across syncs.

```json
{
  "title": "Disc Golf",
  "description": "<p>Premium disc golf tees for players who live for the chains.</p>",
  "rules": [
    { "column": "tag", "relation": "equals", "condition": "niche:disc-golf" }
  ]
}
```

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `title` | Yes | — | Collection title shown to customers |
| `description` | No | `""` | HTML description for the collection page (good for SEO) |
| `rules` | Yes | — | Smart collection rules. Each rule: `{ column, relation, condition }`. Most common: `{ "column": "tag", "relation": "equals", "condition": "niche:<name>" }` |
| `disjunctive` | No | `false` | `true` = match ANY rule, `false` = match ALL rules |

Products are automatically added to collections by Shopify when their tags match the rules — no manual assignment needed.

## How sync works

On merge to `main`, the platform:

1. Diffs the commit to detect which product folders and collection files changed
2. **Collections** (processed first so new products land in correct collections):
   - New `.json` files → creates Shopify smart collection with rules and publishes to all channels
   - Modified `.json` files → updates title, description, and rules
   - Deleted `.json` files → deletes the Shopify collection
3. **New products:**
   - Fetches the Printful catalog to resolve size/color for each variant ID
   - Creates the Shopify product with variants, metafields, and pricing
   - Sets product category automatically from Printful type (e.g., T-SHIRT → Apparel > Clothing > T-Shirts)
   - Publishes to all sales channels (Online Store, Shop app, POS)
   - Links each variant to Printful with the design file
   - Generates mockup images and uploads them to Shopify
3. **Config changes** (product.json modified):
   - Updates title, tags, and variant prices on Shopify
   - No Printful calls — fast
4. **Design changes** (image files modified):
   - Re-links variants to Printful with new design files
   - Regenerates mockup images
5. **Deletions** (folder removed):
   - Deletes the product from Shopify
6. **Meta catalog sync** (runs once at the end):
   - Batches all `custom_label_0`–`4`, `custom_number_0`–`4`, and `internal_label` values from created/updated products into a single Meta Catalog Batch API call
   - Only updates fields that are present in `product.json` — existing Meta data is never deleted
   - `internal_label` controls which Meta product sets a product belongs to (tested/testing/winner)

The repo is the source of truth. Whatever is in `main` is what appears on the Shopify store.
