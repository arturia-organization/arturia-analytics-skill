# Products (Product Master Data)

## Unified Products Table

**Table**: `arturia-bi-405110.dataset.webstore__model_products`
**Purpose**: Product master data with cross-system mapping

### Key Fields

| Field | Type | Description | Analysis Use |
|-------|------|-------------|--------------|
| `product_id` | STRING | Product identifier (`ws_*` or `nwa_*`) | Product joins |
| `product_type_description` | STRING | Category (software/hardware/other) | Category analysis |
| `nwa_product_id` | STRING | New webstore product mapping | Cross-system mapping |

## Legacy VirtueMart Products

**Table**: `arturia-bi-405110.dataset.webstore__model_virtuemart_products`
**Access via**: `product_id` with `ws_` prefix in combined table

## New Webstore Products

**Table**: `arturia-bi-405110.dataset.webstore__model_myarturia_products`
**Key Field**: `uid` (maps to `nwa_product_id` in unified table)
**Access via**: `product_id` with `nwa_` prefix in combined table

## ID Prefixes

| Prefix | System | Example |
|--------|--------|---------|
| `ws_` | VirtueMart (legacy) | `ws_12345` |
| `nwa_` | New Webstore | `nwa_67890` |

## Product Categories

Common values for `product_type_description`:
- `software` - Software instruments and effects
- `hardware` - Physical products (keyboards, controllers)
- `other` - Bundles, upgrades, accessories

## Common Queries

### Products by Category
```sql
SELECT
  product_type_description,
  COUNT(*) as product_count
FROM `arturia-bi-405110.dataset.webstore__model_products`
GROUP BY product_type_description;
```

### Cross-System Product Mapping
```sql
SELECT
  product_id,
  nwa_product_id,
  product_type_description
FROM `arturia-bi-405110.dataset.webstore__model_products`
WHERE nwa_product_id IS NOT NULL;
```
