# Licenses Ranked (First Purchase Identification)

**Table**: `arturia-bi-405110.dataset.webstore__model_licenses_ranked`
**Purpose**: Identifies first product purchase per user chronologically

## Key Fields

| Field | Type | Description | Analysis Use |
|-------|------|-------------|--------------|
| `user_id` | STRING | User identifier | User analysis |
| `product_id` | STRING | Product identifier | Product analysis |
| `product_display_name` | STRING | Human-readable product name | Cohort naming |
| `registered_on` | TIMESTAMP | Registration/purchase timestamp | First purchase date |
| `row_number` | INT | Ranking (1 = first product) | First purchase filter |

## Standard Filter

```sql
WHERE row_number = 1
  AND user_id > 0
  AND product_id IS NOT NULL
```

## Common Queries

### First Purchase Cohorts
```sql
SELECT
  product_display_name as first_product,
  COUNT(DISTINCT user_id) as users
FROM `arturia-bi-405110.dataset.webstore__model_licenses_ranked`
WHERE row_number = 1
  AND user_id > 0
GROUP BY first_product
ORDER BY users DESC;
```

### First Purchase by Month
```sql
SELECT
  DATE_TRUNC(registered_on, MONTH) as month,
  COUNT(DISTINCT user_id) as new_customers
FROM `arturia-bi-405110.dataset.webstore__model_licenses_ranked`
WHERE row_number = 1
  AND user_id > 0
GROUP BY month
ORDER BY month;
```

### Join with Order Data for Revenue Analysis
```sql
WITH first_products AS (
  SELECT user_id, product_id, product_display_name
  FROM `arturia-bi-405110.dataset.webstore__model_licenses_ranked`
  WHERE row_number = 1 AND user_id > 0
)
SELECT
  fp.product_display_name as first_product,
  COUNT(DISTINCT o.virtuemart_user_id) as customers,
  SUM(o.product_final_price) as total_revenue
FROM first_products fp
JOIN `arturia-bi-405110.dataset.webstore__model_combined_order_items` o
  ON fp.user_id = o.virtuemart_user_id
WHERE o.order_status IN ('C', 'D', 'S', 'U', 'Shipped', 'Completed', 'Confirmed')
  AND o.product_final_price > 0
GROUP BY first_product
ORDER BY total_revenue DESC;
```

## Use Cases

- **Cohort Analysis**: Group users by their first purchased product
- **Customer Journey**: Understand entry points into the Arturia ecosystem
- **Product Affinity**: Analyze which products lead to further purchases
