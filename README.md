# dbt Data Transformation Pipeline for Databricks

## Overview
A dbt project on Databricks that transforms raw retail sales data (sales, returns, customers, products, stores, dates) into clean, tested, analytics-ready tables using a bronze → silver → gold layered architecture.

Built while learning dbt fundamentals through a structured video tutorial, then used to practice core dbt concepts hands-on: sources, custom macros, custom generic tests, and snapshots for slowly changing dimensions.

## Architecture

```
Databricks source tables (fact_sales, fact_returns, dim_customer,
dim_date, dim_product, dim_store, items)
        │
        ▼
   bronze/   — 1:1 views/tables over source data, schema: bronze
        │
        ▼
   silver/   — joined, aggregated business logic, schema: silver
        │
        ▼
   gold/     — deduplicated, snapshot-tracked entities, schema: gold
```
  
Each layer is materialized into its own Databricks schema (`bronze`, `silver`, `gold`), configured centrally in `dbt_project.yml` rather than per-model, except where a model overrides it (e.g. `bronze_product` is a `view` while the rest of bronze defaults to `table`).

## Models

| Model | Layer | What it does |
|---|---|---|
| `bronze_customer`, `bronze_date`, `bronze_product`, `bronze_store`, `bronze_sales`, `bronze_returns` | bronze | Direct pass-through from Databricks sources, one model per source table |
| `silver_salesinfo` | silver | Joins sales to products and customers, computes `calculated_gross_amount` via a custom `multiply()` macro, aggregates total sales by product category and customer gender |
| `source_gold_items` | gold | Deduplicates the `items` source using `row_number() over (partition by id order by updateDate desc)`, keeping only the latest record per `id` |
| `gold_items` (snapshot) | gold | Tracks historical changes to `source_gold_items` over time using dbt's timestamp-based snapshot strategy (SCD Type 2), keyed on `id`, watching `updateDate` |

## Tests
- **`bronze_sales.sales_id`** — `unique` + `not_null`
- **`bronze_sales.gross_amount`** — custom generic test `generic_non_negative`, which flags any row where the column is negative
- **`bronze_store.store_sk`** — `unique` + `not_null`
- **`bronze_store.store_name`** — `accepted_values` against a fixed list of 5 store names, set to `severity: warn` (flags anomalies without failing the build)
- A singular test (`non_negative_test.sql`) checks that `bronze_sales` never has both `gross_amount` and `net_amount` negative simultaneously

## Macros
- `multiply(col1, col2)` — a small reusable macro for `col1 * col2`, used in `silver_salesinfo` to compute gross amount from unit price and quantity
- `generate_schema_name` — overrides dbt's default schema-naming behavior so models land in exactly the schema configured (`bronze`/`silver`/`gold`) rather than dbt's default `<target_schema>_<custom_schema>` pattern

## Environment & Tooling
- **uv** for dependency management — `uv.lock` pins exact package versions so the project runs identically across machines
- **pyproject.toml** — Python project metadata and dependencies
- Databricks SQL Warehouse as the dbt target, via `profiles.yml`

## How to Run
```bash
uv sync
cd dbt_dev
dbt debug        # verify connection to Databricks
dbt seed         # load the lookup seed table
dbt run
dbt test
dbt snapshot      # capture SCD2 history for gold_items
```

## Credit
Built while following a structured dbt + Databricks video tutorial to learn dbt fundamentals, then used to practice the concepts directly (macros, custom tests, snapshots) rather than as a from-scratch design.

