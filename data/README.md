# Data

No raw data or database files are committed here. The dashboard reads the **synthetic BeanStream
warehouse** from:

[`github.com/malikb0/ecommerce-data-platform-practice-project`](https://github.com/malikb0/ecommerce-data-platform-practice-project)

## Connection (defaults)

| Setting | Value |
|---------|-------|
| Server | `localhost` |
| Port | `5434` |
| Database | `warehouse` |
| User | `learner` |
| Password | `learner` |

These are the defaults in the platform repo's `docker-compose.yml` / `.env.example`; use your `.env`
values if you changed them.

## Rebuild the warehouse

In the platform repo:

```bash
make up      # PostgreSQL :5434 + MySQL :3308
make seed    # load synthetic CSVs into MySQL
make etl     # build the star schema (offline SQLite + live PostgreSQL)
```

The compose project is named `ecommerce-data-platform-practice-project` (the `name:` field), so the
derived containers are:

- `ecommerce-data-platform-practice-project-postgres-1`
- `ecommerce-data-platform-practice-project-mysql-1`

Confirm with `docker compose ps`. **Power BI connects to `localhost:5434`, not to the container name.**

## Tables

| Table | Grain | Columns |
|-------|-------|---------|
| `fact_sales` | one row per order line | `order_item_id`, `order_id`, `date_id`, `customer_id`, `product_id`, `channel_id`, `order_status`, `quantity`, `unit_price`, `line_amount` |
| `dim_date` | one row per date | `date_id` (YYYYMMDD), `full_date`, `year`, `quarter`, `month`, `month_name`, `day`, `day_of_week`, `day_name`, `is_weekend` |
| `dim_customer` | one row per customer | `customer_id`, `full_name`, `email`, `city`, `country`, `signup_date` |
| `dim_product` | one row per product | `product_id`, `product_name`, `category`, `unit_price` |
| `dim_channel` | one row per channel | `channel_id`, `channel` |
| `etl_watermark` | ETL bookkeeping | **do not load** |

Column types are set in Power Query; see [the build guide §4](../docs/spec.md#4-clean-in-power-query).

## Grain and rules

- `fact_sales` is **one row per order line** (`order_item_id`).
- `line_amount = quantity × unit_price`.
- `order_status` ∈ `{completed, shipped, pending, cancelled}`.
- **Cancelled orders stay in the fact table** (so status is analysable) but are excluded from revenue
  **in the measures** — do not filter them out in Power Query.
- All data is synthetic, generated with seed 42 by the platform repo. Nothing is downloaded.

## Not in this dataset

There are no delivery dates, promised dates, invoices, or payments. **On-time rate** and **receivables
ageing** are out of scope; extend the generator first if they are ever needed. Do not fake them.
