# Arturia BI Skill

A Claude skill for analyzing Arturia's BigQuery data lake, covering webstore transactions and ASC telemetry.

## Installation

### Claude Code

```bash
# Add to your project
cp -r . ~/.claude/skills/arturia-bi/

# Or clone directly
git clone https://github.com/arturia-organization/arturia-analytics-skill.git ~/.claude/skills/arturia-bi
```

### Claude.ai

Enable via Settings or ask your team admin to authorize organization-wide access.

## What's Included

| Table Documentation | Description |
|---------------------|-------------|
| [asc_model_stats](tables/asc_model_stats.md) | Plugin usage telemetry (152M+ rows) |
| [combined_order_items](tables/combined_order_items.md) | Unified order/revenue data |
| [licenses_ranked](tables/licenses_ranked.md) | First purchase identification |
| [products](tables/products.md) | Product master data |
| [users](tables/users.md) | User profiles |

## Usage

Once installed, Claude will automatically use this skill when you ask about:

- Writing BigQuery SQL queries
- Analyzing orders, revenue, or telemetry
- Understanding table schemas and columns
- Applying correct filters and avoiding data quality pitfalls

## Examples

- "Write a query for daily active users"
- "How do I calculate revenue by country?"
- "What's the correct timestamp field for telemetry?"
- "Show me the order status values for completed orders"

## Dataset

`arturia-bi-405110.dataset`

## Key Warnings

- Use `original_session_timestamp` for telemetry (not `session_timestamp`)
- Include all order statuses: `'C','D','S','U','Shipped','Completed','Confirmed'`
- Filter `user_id > 0` to exclude system users
