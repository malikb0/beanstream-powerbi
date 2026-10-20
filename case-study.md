# Case study — Power BI KPI dashboard for BeanStream

**Role:** Data / BI · **Tools:** Power BI Desktop, Power Query, DAX, PostgreSQL ·
**Status:** Built 2026-10-20

![Executive dashboard](screenshots/01-executive.png)

## Context

BeanStream is a fictional coffee-subscription retailer. I had already built its end-to-end data
platform — synthetic CSVs → MySQL OLTP → star-schema PostgreSQL warehouse → incremental, watermarked
ETL, with 24 tests. The platform shipped SQL reports and a KPI summary, but no interactive BI layer.

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
|-----|------:|
| Revenue (non-cancelled) | 7,261.50 |
| Orders | 99 |
| Units | 448 |
| AOV | 73.35 |
| Customers | 40 |
| Products | 12 |

Revenue peaks in October at $1,273.00; the lowest month is March ($252.75), with a sharp November
drop to $286.00. Coffee is the largest category and the app channel contributes about half of
revenue; AOV is $73.35 across 99 orders.

## What I'd do next

Incremental refresh, row-level security by country, and a published report with a scheduled refresh.

## Links

- Repo: [github.com/malikb0/beanstream-powerbi](https://github.com/malikb0/beanstream-powerbi)
- Platform: [ecommerce-data-platform-practice-project](https://github.com/malikb0/ecommerce-data-platform-practice-project)
