---
name: arturia-bi
description: Analyze Arturia webstore and telemetry data in BigQuery. Use when writing SQL queries, analyzing orders/revenue, telemetry/usage data, or asking about table schemas, columns, filters, and data quality.
allowed-tools: Read, Grep, Glob, Bash, mcp__bigquery
---

# Arturia BI

## Purpose
Assist with data analysis on Arturia's BigQuery data lake, covering webstore transactions and ASC telemetry.

## Query Execution

**Check for `bq` CLI first**, then fall back to BigQuery MCP:

1. **If `bq` CLI is available** (Claude Code): Use Bash with `bq query`
   ```bash
   bq query --use_legacy_sql=false --max_rows=1000 "SELECT ... FROM \`arturia-bi-405110.dataset.table\`"
   ```

2. **If MCP is available** (Claude.ai or no CLI): Use the `mcp__bigquery` tool

3. **Otherwise**: Provide the SQL query for the user to run manually

## Instructions

1. **Read the relevant table documentation** from `tables/` folder for detailed schema info
2. **Use correct timestamp fields**:
   - ✅ `original_session_timestamp` for telemetry analysis
   - ❌ Never use `session_timestamp` (unreliable hybrid field)
3. **Apply standard filters** for valid data:
   - Telemetry: `user_id > 0 AND session_duration > 0`
   - Revenue: `order_status IN ('C','D','S','U','Shipped','Completed','Confirmed') AND product_final_price > 0`
4. **Warn about known data quality issues** documented in each table file

## Table Documentation

| Table | Doc | Purpose |
|-------|-----|---------|
| `asc__model_stats` | [tables/asc_model_stats.md](tables/asc_model_stats.md) | Plugin usage telemetry (152M+ rows) |
| `webstore__model_combined_order_items` | [tables/combined_order_items.md](tables/combined_order_items.md) | Unified order data |
| `webstore__model_licenses_ranked` | [tables/licenses_ranked.md](tables/licenses_ranked.md) | First purchase identification |
| `webstore__model_products` | [tables/products.md](tables/products.md) | Product master data |
| `webstore__model_users` | [tables/users.md](tables/users.md) | User profiles |

## Common Tasks
- Daily/monthly active users
- Revenue analysis by product, region, time
- Product usage patterns
- First purchase cohorts
- DAW and plugin format distribution

## Dataset
`arturia-bi-405110.dataset`

## Table Relationships

```
asc__model_stats (telemetry)
    ↓ user_id
webstore__model_users (user profiles)
    ↑ virtuemart_user_id
webstore__model_combined_order_items (all orders)
    ↓ product_id
webstore__model_products (product details)

webstore__model_licenses_ranked (first purchases)
    ↓ user_id
webstore__model_combined_order_items (all orders)
```
