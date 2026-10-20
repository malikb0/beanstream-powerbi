# Case study — Power BI KPI dashboard for BeanStream

BeanStream is a fictional coffee-subscription retailer. The data platform already existed; this project
adds the missing BI layer — a three-page Power BI report over the warehouse, with 16 DAX measures and an
exact reconciliation to the source KPIs.

![Executive dashboard](screenshots/01-executive.png)

## At a glance

| Field | Detail |
|---|---|
| **Role** | Data / BI |
| **Tools** | Power BI Desktop · Power Query · DAX · PostgreSQL |
| **Data** | Synthetic BeanStream warehouse (2024, seed 42) |
| **Scope** | 3 pages · 16 measures · 5 tables |
| **Status** | Built 2026-10-20 |
| **Result** | Reconciles exactly to the source KPI summary |

## Context

I had already built BeanStream's end-to-end data platform: synthetic CSVs → MySQL OLTP →
star-schema PostgreSQL warehouse → incremental, watermarked ETL, with 24 tests. The platform shipped SQL
reports and a KPI summary, but no interactive BI layer.

## Problem

The SQL reports answered fixed questions. Stakeholders needed to slice revenue by period, category,
channel, and country without writing SQL — and I needed to close a Power BI gap (the top BI tool in
the local data market).

## Build

- Connected Power BI Desktop to the PostgreSQL warehouse in Import mode.
- Cleaned and typed five tables in Power Query; kept cancelled orders in the fact table.
- Modelled a star schema (one fact, four dimensions) with single-direction relationships, a marked
  date table, and sorted month/day names.
- Wrote 16 DAX measures, including MoM/YoY, a latest-month MoM, a running total, cancellation rate, and
  Top-N ranking.
- Built three pages: Executive, Trends, Products & Customers.

## Technical highlights

- Import-mode star schema: one fact, four dimensions, four single-direction relationships.
- `MonthStart` date column for reliable `DATEADD`-based MoM; a latest-month measure for the KPI card.
- 16 measures across **Core**, **Time**, **Quality**, and **Share** — see the
  [measure catalogue](docs/measures.md).
- Cancelled orders excluded in measures, visible as a 17.5% quality KPI.
- Accessibility: alt text, tab order, no 3D, page-only screenshots.
- Documentation covers architecture, grain, DAX, validation, and the rebuild path — see the
  [documentation hub](docs/README.md).

## Decisions

- **Cancelled orders stay in the fact table** and are excluded in measures, so cancellation remains
  analysable. Filtering them in Power Query would have hidden a real business signal.
- **Single-direction relationships** avoid ambiguous filter paths.
- **Measures over calculated columns** keep the model small and the logic reusable.
- **No on-time or receivables metrics** — the dataset has no delivery or invoice fields. I left them
  out rather than inventing them.
- **YoY is modelled but not shown** — the synthetic dataset covers one year (2024), so prior-year
  revenue is blank. I kept the measure and left it off the page rather than showing an empty visual.

## Result

The report reconciles exactly to the platform's KPI summary:

| KPI | Value |
|---|---:|
| Revenue (non-cancelled) | 7,261.50 |
| Orders | 99 |
| Units | 448 |
| AOV | 73.35 |
| Active customers | 39 |
| Customer dimension rows | 40 |
| Products | 12 |
| Cancellation rate | 17.5% |
| Latest month MoM | +190.03% |

Revenue peaks in October at $1,273.00; the lowest month is March ($252.75), with a sharp November drop
to $286.00. Coffee is the largest category and the app channel contributes about half of revenue; AOV is
$73.35 across 99 orders.

## The report

| Executive | Trends | Products & Customers |
|---|---|---|
| <a href="screenshots/01-executive.png"><img src="screenshots/01-executive.png" width="260" alt="Executive page"></a> | <a href="screenshots/02-trends.png"><img src="screenshots/02-trends.png" width="260" alt="Trends page"></a> | <a href="screenshots/03-products.png"><img src="screenshots/03-products.png" width="260" alt="Products and Customers page"></a> |

The Executive page answers *how much and where*; Trends answers *how it is moving*; Products &
Customers answers *what drives revenue*. Click any page for the full-size PNG.

## Evidence

| What | Where |
|---|---|
| KPI reconciliation | [README validation table](README.md) and [build guide §12](docs/spec.md#12-reconcile) |
| Measure definitions | [`dax/measures.dax`](dax/measures.dax) and [measure catalogue](docs/measures.md) |
| Model design | [Data model](docs/data-model.md) and [architecture](docs/architecture.md) |
| Rebuild path | [Build guide](docs/spec.md) |

## What I'd do next

Incremental refresh, row-level security by country, and a published report with a scheduled refresh.

## Links

- Repo: [github.com/malikb0/beanstream-powerbi](https://github.com/malikb0/beanstream-powerbi)
- Platform: [ecommerce-data-platform-practice-project](https://github.com/malikb0/ecommerce-data-platform-practice-project)
- Screenshots: [Executive](screenshots/01-executive.png) · [Trends](screenshots/02-trends.png) ·
  [Products & Customers](screenshots/03-products.png)
