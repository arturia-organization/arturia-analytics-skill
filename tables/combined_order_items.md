# Combined Order Items (Primary Transaction Table)

**Table**: `arturia-bi-405110.dataset.webstore__model_combined_order_items`
**Purpose**: Unified view of all order items from both VirtueMart (legacy) and NWA (new) webstore systems
**Primary Key**: `order_item_id`

## Key Fields

| Field | Type | Description | Analysis Use |
|-------|------|-------------|--------------|
| `order_item_id` | STRING | Unique identifier (`ws_*` for VM, `nwa_*` for NWA) | Primary key for joins |
| `virtuemart_user_id` | STRING | Unified user identifier across both systems | Customer analysis, cohorts |
| `order_id` | STRING | Order identifier (`ws_*` for VM, correlation_id for NWA) | Order-level analysis |
| `order_number` | STRING | Human-readable order number (truncated before '_') | Order tracking |
| `product_id` | STRING | Product identifier (`ws_*` for VM, `nwa_*` for NWA) | Product analysis |
| `product_final_price` | DECIMAL | **CRITICAL**: Final price after all adjustments | Revenue calculations |
| `product_quantity` | INT | Quantity ordered | Volume analysis |
| `order_status` | STRING | **CRITICAL**: Order completion status | Revenue filtering |
| `currency_code` | STRING | ISO 4217 currency code (EUR, USD, etc.) | Multi-currency analysis |
| `created_on` | TIMESTAMP | Order creation timestamp (UTC) | Temporal analysis |
| `product_type_description` | STRING | Product category (software/hardware/other) | Category analysis |
| `first_order_indicator` | INT | Row number for first purchase per product type | First purchase identification |
| `city` | STRING | Customer city | Geographic analysis |
| `country` | STRING | Customer country | Geographic analysis |
| `continent` | STRING | Customer continent | Regional analysis |

## Order Status Values

### VirtueMart (Legacy) - Completed Statuses
| Status | Description |
|--------|-------------|
| `'C'` | Confirmed (primary) |
| `'D'` | Re-Deliver missing Licences |
| `'S'` | Shipped |
| `'U'` | Confirmed by shopper |

### NWA (New Webstore) - Completed Statuses
| Status | Description |
|--------|-------------|
| `'Completed'` | Primary status (99.4% of new orders) |
| `'Confirmed'` | Secondary status |
| `'Shipped'` | Physical fulfillment |

**⚠️ CRITICAL**: Always include both `'Completed'` and `'Shipped'` for new webstore data.

## Standard Revenue Filter

```sql
WHERE order_status IN ('C', 'D', 'S', 'U', 'Shipped', 'Completed', 'Confirmed')
  AND product_final_price > 0
  AND virtuemart_user_id > 0
```

## ID Prefixes

| Prefix | System | Period |
|--------|--------|--------|
| `ws_` | VirtueMart (legacy) | Through Dec 2024 |
| `nwa_` | New Webstore | Jan 2025+ |

## Common Queries

### Total Revenue
```sql
SELECT
  SUM(product_final_price) as total_revenue,
  COUNT(DISTINCT virtuemart_user_id) as unique_customers,
  COUNT(DISTINCT order_id) as total_orders
FROM `arturia-bi-405110.dataset.webstore__model_combined_order_items`
WHERE order_status IN ('C', 'D', 'S', 'U', 'Shipped', 'Completed', 'Confirmed')
  AND product_final_price > 0
  AND virtuemart_user_id > 0;
```

### Revenue by Product Category
```sql
SELECT
  product_type_description,
  COUNT(*) as order_count,
  SUM(product_final_price) as revenue
FROM `arturia-bi-405110.dataset.webstore__model_combined_order_items`
WHERE order_status IN ('C', 'D', 'S', 'U', 'Shipped', 'Completed', 'Confirmed')
  AND product_final_price > 0
GROUP BY product_type_description;
```

### Revenue by Geography
```sql
SELECT
  country,
  continent,
  COUNT(DISTINCT virtuemart_user_id) as customers,
  SUM(product_final_price) as revenue
FROM `arturia-bi-405110.dataset.webstore__model_combined_order_items`
WHERE order_status IN ('C', 'D', 'S', 'U', 'Shipped', 'Completed', 'Confirmed')
  AND product_final_price > 0
GROUP BY country, continent
ORDER BY revenue DESC;
```

### Monthly Revenue Trend
```sql
SELECT
  DATE_TRUNC(created_on, MONTH) as month,
  SUM(product_final_price) as revenue,
  COUNT(DISTINCT virtuemart_user_id) as customers
FROM `arturia-bi-405110.dataset.webstore__model_combined_order_items`
WHERE order_status IN ('C', 'D', 'S', 'U', 'Shipped', 'Completed', 'Confirmed')
  AND product_final_price > 0
GROUP BY month
ORDER BY month;
```

## Common Pitfalls

- ❌ **Missing New Webstore Status**: Only using `'Shipped'` instead of `'Completed'`
- ❌ **Wrong User Field**: Using `user_id` instead of `virtuemart_user_id`
- ❌ **Incomplete Product Mapping**: Not handling `ws_`/`nwa_` prefixes correctly
- ❌ **Currency Assumptions**: Treating all revenue as same currency (no normalization applied)

## System Transition Timeline

| Period | System | Primary Status |
|--------|--------|----------------|
| Through Dec 2024 | VirtueMart | `'C'` (Confirmed) |
| Jan 2025+ | NWA | `'Completed'` |
| Transition | Both | Mixed statuses |
