# Amazon Sales E-Commerce ETL Pipeline (Azure Databricks + PySpark)

An end-to-end data engineering pipeline that ingests, cleans, and models e-commerce sales data using the **Medallion Architecture (Bronze → Silver → Gold)** on Databricks, culminating in a star-schema data model ready for Power BI reporting.

## Overview

This project simulates a real-world e-commerce analytics pipeline for an Amazon-style sales dataset. Raw transactional and reference data is ingested, cleaned, and transformed through three layers, then modeled into fact and dimension tables for business intelligence reporting.

Both a **full-load** pipeline (initial historical load) and an **incremental-load** pipeline (ongoing daily ingestion) are implemented, reflecting how production data pipelines typically operate.

## Architecture

```
Raw CSV files (landing zone)
        │
        ▼
   BRONZE LAYER   →  Raw ingestion, schema enforcement, file archiving, audit columns
        │
        ▼
   SILVER LAYER   →  Data cleaning, type casting, deduplication, business rule fixes
        │
        ▼
   GOLD LAYER     →  Star schema (fact + dimension tables), currency conversion,
                      BI-ready aggregates
        │
        ▼
   Power BI Dashboard
```

**Tech stack:** Azure Databricks, PySpark, Delta Lake, Delta Change Data Feed, Python, SQL, Power BI

## Data Model

**Fact table:**
- `gld_fact_order_items` — order line-item transactions (quantity, pricing, discounts, tax, net amount in original currency and INR)

**Dimension tables:**
- `gld_dim_products` — products joined with brand and category
- `gld_dim_customers` — customers enriched with a country/state → region mapping
- `gld_dim_date` — calendar dimension with year, month, quarter, week, weekend flag

This forms a standard **star schema**, with `gld_fact_order_items` as the central fact table joined to the dimension tables via `date_id`, `customer_id`, and `product_id`.

## Pipeline Details

### Bronze Layer — Raw Ingestion
- Enforces explicit schemas on read (`StructType`) rather than relying on schema inference, for reliability and consistency
- Adds audit columns: `_source_file`, `_ingested_at`/`ingest_timestamp` for traceability
- Archives processed source files into a dated folder structure after successful load, preventing reprocessing
- Fact table pipeline supports **incremental loads** using `append` mode with **Delta Change Data Feed** enabled, so downstream consumers can track row-level changes

### Silver Layer — Cleaning & Transformation
Real data quality issues were identified and resolved, including:
- Type casting: strings → integers, doubles, dates, and timestamps, with fallback date-format parsing
- Currency symbol and percentage sign stripping (e.g. `"$45.00"` → `45.00`, `"21%"` → `21.0`)
- Text normalization: category code typos (e.g. `GROCERY` → `GRCY`), inconsistent casing, spelling corrections (`"Coton"` → `"Cotton"`, `"Alumium"` → `"Aluminum"`)
- Null handling: dropped records with missing keys, imputed defaults for missing contact fields
- Deduplication on natural keys (e.g. `order_id` + `item_seq`, `date`, `category_code`)
- Business logic fixes: correcting negative values, standardizing channel labels (`web` → `Website`, `app` → `Mobile`)

### Gold Layer — BI-Ready Modeling
- Computes derived business metrics: `gross_amount`, `discount_amount`, `net_amount`
- Applies a **multi-currency conversion** to a common reporting currency (INR) using a fixed FX rate table joined against transaction currency
- Builds the **customer region dimension** by mapping country + state combinations (across multiple countries) to sales regions via a custom lookup table
- Builds the **date dimension** with derived attributes (month name, quarter label, week label, weekend flag) for time-intelligence in Power BI
- Writes all tables in Delta format with schema evolution support (`mergeSchema`)

## Repository Structure

```
├── 1_dim_bronze.ipynb          # Ingest dimension source data (brands, category, products, customers, calendar)
├── 2_dim_silver.ipynb          # Clean & transform dimension data
├── 3_dim_gold.ipynb            # Build dimension tables (star schema)
├── 1_fact_bronze.ipynb         # Full historical load of fact data (order items)
├── 2_fact_silver.ipynb         # Clean & transform fact data
├── 3_fact_gold.ipynb           # Build fact table with derived metrics + currency conversion
├── 1_inc_fact_bronze.ipynb     # Incremental (daily) load of new fact data
├── 2_inc_fact_silver.ipynb     # Incremental cleaning & transformation
├── 3_inc_fact_gold.ipynb       # Incremental gold table updates
└── dashboard/                  # Power BI dashboard screenshots
```

## Power BI Dashboard

Dashboards were built on top of the Gold layer tables to surface sales trends, regional performance, and discount/channel analysis.

*(Add dashboard screenshot(s) here — see `/dashboard` folder)*

## What This Project Demonstrates

- Designing and implementing a Medallion Architecture from raw files to BI-ready tables
- Writing production-style PySpark code: schema enforcement, incremental loading, Delta Lake features (Change Data Feed, schema evolution)
- Realistic data cleaning: handling messy real-world issues (typos, inconsistent formats, nulls, negative values, multi-currency data)
- Dimensional data modeling (star schema) for analytics
- Bridging the gap between raw data and business reporting (Power BI)

## Author

**Vinod Kumar Vasantha**
Aspiring Azure Data Engineer | [LinkedIn](#)
