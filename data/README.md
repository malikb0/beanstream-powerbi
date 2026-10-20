# Data dictionary

The dashboard is only as trustworthy as the warehouse beneath it. This page describes the synthetic
BeanStream data the report reads, how to rebuild it, which tables are loaded, and the business rules
the measures depend on.

No raw CSVs, database dumps, or warehouse files are committed in this repository. The report connects
to the PostgreSQL warehouse owned by the separate
[`ecommerce-data-platform-practice-project`](https://github.com/malikb0/ecommerce-data-platform-practice-project) repository.

## At a glance

| Item | Detail |
|---|---|
| Source | Synthetic BeanStream commerce data, generated with seed 42 |
| Upstream | MySQL OLTP → incremental ETL → PostgreSQL star-schema warehouse |
| Database | `warehouse` on `localhost:5434` |
| Fact grain | One row per order line (`order_item_id`) |
| Coverage | Calendar year 2024 |
| Loaded tables | Five: `fact_sales` + four dimensions |
| Report mode | Power BI **Import** |
| Data status | Synthetic only; no real customers, orders, or payments |

## Where the data comes from

The platform repository owns the whole upstream path:

```text
synthetic generator → MySQL OLTP (:3308) → incremental ETL → PostgreSQL warehouse (:5434) → Power BI
```

This repository begins at the last arrow. It does not generate the source files or run the ETL. If the
warehouse is missing or stale, rebuild it in the platform repo before refreshing the report.

## Connection defaults

| Setting | Value |
|---|---|
| Server | `localhost` |
| Port | `5434` |
| Database | `warehouse` |
| User | `learner` |
| Password | `learner` |

These are the platform repo's defaults in `docker-compose.yml` and `.env.example`. If you changed your
`.env`, use those values in Power BI instead. **Connect to `localhost:5434`, not to the container
name.**

## Rebuild the warehouse

Run these commands in the **platform repository**, not in this one:

```bash
make setup   # venv + dependencies + synthetic data
make up      # PostgreSQL :5434 + MySQL :3308
make seed    # load synthetic CSVs into MySQL
make etl     # build the star schema (offline SQLite + live PostgreSQL)
```

`make setup` is a first-run step. Later rebuilds normally need `make up`, `make seed`, and `make etl`.

The compose project is named `ecommerce-data-platform-practice-project`, so the derived containers
are:

- `ecommerce-data-platform-practice-project-postgres-1`
- `ecommerce-data-platform-practice-project-mysql-1`

Confirm both are running with `docker compose ps`. PostgreSQL should report healthy before you open
Power BI.

## What gets loaded

| Table | Role | Grain | Report use |
|---|---|---|---|
| `fact_sales` | Fact | One row per order line | Revenue, orders, units, time trends, product/customer rankings |
| `dim_date` | Dimension | One row per calendar date | Date table, monthly axes, month-over-month logic |
| `dim_customer` | Dimension | One row per customer | Customer counts, country analysis, Top 10 customers |
| `dim_product` | Dimension | One row per product | Category analysis, Top 5 products |
| `dim_channel` | Dimension | One row per channel | Channel mix and slicers |
| `etl_watermark` | ETL bookkeeping | Last successful load marker | **Do not load** |

The five loaded tables keep their exact source names; the DAX references them by name. Column types
are set in Power Query; the full type table is in [the build guide §4](../docs/spec.md#4-clean-in-power-query).

## Grain and business rules

- `fact_sales` is **one row per order line**, not one row per order.
- `line_amount = quantity × unit_price`.
- `order_status` is one of `completed`, `shipped`, `pending`, or `cancelled`.
- **Cancelled orders stay in the fact table.** They are excluded from revenue, order, unit, and
  customer measures inside DAX — not filtered out in Power Query. That keeps cancellation analysable.
- All data is synthetic, generated with seed 42. Nothing is downloaded or scraped.

> **Why keep cancelled orders?** Filtering them upstream would hide a real business signal. The report
> can show a **17.5% cancellation rate** while the top-line revenue cards remain clean. See
> [Architecture](../docs/architecture.md#cancelled-orders-remain-in-the-fact).

## Expected checks

Use these figures to confirm the warehouse before refreshing the report:

| Check | Expected |
|---|---:|
| All orders | **120** |
| Non-cancelled orders | **99** |
| Revenue (non-cancelled) | **$7,261.50** |
| Units sold (non-cancelled) | **448** |
| Cancelled orders | **21** (**17.5%** of all orders) |
| Active customers in the fact | **39** |
| Rows in `dim_customer` | **40** |
| Rows in `dim_product` | **12** |

The difference between **39 active customers** and **40 dimension rows** is expected: one customer has
no non-cancelled fact activity. The report uses the measure, not the dimension row count.

The source of these numbers is the platform repo's
[`kpi_summary.md`](https://github.com/malikb0/ecommerce-data-platform-practice-project/blob/main/reports/expected/kpi_summary.md).
The report-side reconciliation is in [Build guide §12](../docs/spec.md#12-reconcile).

## What this dataset does not contain

There are no delivery dates, promised dates, invoices, or payments. **On-time rate** and **receivables
ageing** are out of scope; extend the generator first if they are ever needed. Do not fake them.

The warehouse covers **2024 only**, so prior-year revenue and YoY are blank by design. Currency is not
encoded in the source data; `$` is a display assumption in the report.

## Related documents

- [Database and model design](../docs/architecture.md)
- [Star schema and relationships](../docs/data-model.md)
- [Measure catalogue](../docs/measures.md)
- [Build guide](../docs/spec.md)
