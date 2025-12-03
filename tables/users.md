# Users (User Profiles)

## Unified Users Table

**Table**: `arturia-bi-405110.dataset.webstore__model_users`
**Purpose**: User profile and geographic data

### Key Fields

| Field | Type | Description | Analysis Use |
|-------|------|-------------|--------------|
| `user_id` | STRING | User identifier | User analysis |
| `city` | STRING | User city | Geographic analysis |
| `country` | STRING | User country (normalized) | Geographic analysis |
| `continent` | STRING | User continent | Regional analysis |

## Data Sources

### Legacy VirtueMart Users
- **Source**: Clean webstore VirtueMart tables
- **Access via**: `virtuemart_user_id` field in order tables

### New Webstore Users
- **Source**: Payment tracker and NWA order addresses
- **Access via**: `user_id` in payment tracker

## Geographic Normalization

Country names are normalized for consistency:
- `'UK'` → `'United Kingdom'`
- Other standard normalizations applied

## Standard Filter

```sql
WHERE user_id > 0  -- Excludes system users
```

## Common Queries

### Users by Country
```sql
SELECT
  country,
  continent,
  COUNT(DISTINCT user_id) as users
FROM `arturia-bi-405110.dataset.webstore__model_users`
WHERE user_id > 0
GROUP BY country, continent
ORDER BY users DESC;
```

### Users by Continent
```sql
SELECT
  continent,
  COUNT(DISTINCT user_id) as users
FROM `arturia-bi-405110.dataset.webstore__model_users`
WHERE user_id > 0
GROUP BY continent
ORDER BY users DESC;
```

## Linking to Other Tables

### Link to Telemetry
```sql
SELECT
  u.country,
  COUNT(DISTINCT t.session_uuid) as sessions
FROM `arturia-bi-405110.dataset.webstore__model_users` u
JOIN `arturia-bi-405110.dataset.asc__model_stats` t
  ON u.user_id = CAST(t.user_id AS STRING)
WHERE u.user_id > 0 AND t.user_id > 0
GROUP BY u.country;
```

### Link to Orders
```sql
SELECT
  u.country,
  SUM(o.product_final_price) as revenue
FROM `arturia-bi-405110.dataset.webstore__model_users` u
JOIN `arturia-bi-405110.dataset.webstore__model_combined_order_items` o
  ON u.user_id = o.virtuemart_user_id
WHERE o.order_status IN ('C', 'D', 'S', 'U', 'Shipped', 'Completed', 'Confirmed')
  AND o.product_final_price > 0
GROUP BY u.country
ORDER BY revenue DESC;
```
