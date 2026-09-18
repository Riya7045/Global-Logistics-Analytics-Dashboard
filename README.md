# Supply Chain & Logistics Performance Dashboard

An end-to-end analytics project that cleans, models, and visualizes 180K+ orders from the DataCo Supply Chain dataset to uncover on-time delivery and SLA performance issues across shipping modes, regions, and product categories.

The pipeline moves data through three stages:

**Python (clean & explore) → SQL Server (star schema & KPI views) → Power BI (dashboard)**

## Key Findings

- **First Class shipping has a 100% SLA breach rate and 0% on-time delivery**, despite being the premium tier — a critical pricing-vs-performance misalignment. Standard Class outperforms every other mode with 57.7% OTD and the lowest breach rate (39.8%).
- **Every region falls below an 85% OTD benchmark.** South of USA ranks 2nd worst globally (39.5% OTD, using the full-dataset view), 6.3 points behind Canada, the best-performing region; Central Africa is worst.
- Overall KPI benchmarks across the full dataset: **OTD 40.9%**, **OTIF 40.9%**, **SLA breach 57.3%**, **Perfect Order rate 14.0%**, **Complete order rate 33.0%**.

## Repository Structure

```
├── Python/          Data cleaning & EDA notebook + generated charts
├── SQL/              T-SQL scripts: staging → star schema → KPI views
├── Power BI file/     .pbix dashboard
├── Excel/             Supporting pivot summary
└── Screenshots/       Dashboard page exports
```

## Pipeline

### 1. Python — Cleaning & EDA
[`Supply chain - Data Cleaning and EDA.ipynb`](Python/Supply%20chain%20-%20Data%20Cleaning%20and%20EDA.ipynb)
- Loads the raw DataCo Supply Chain dataset (180,519 rows × 53 columns)
- Selects 25 relevant columns, drops PII, renames to snake_case for SQL Server
- Fixes data types (dates, numerics) and extracts date parts (year, month, quarter, day of week)
- Engineers KPI flags: `is_on_time`, `is_otif`, `is_sla_breach`, `is_perfect_order`, `is_complete`, `days_variance`
- Produces exploratory charts: delay by shipping mode, delay by region, SLA breach heatmap (region × shipping mode)
- Exports a clean, typed CSV ready for SQL Server import

### 2. SQL Server — Star Schema & KPI Views
Scripts run in order under [`SQL/`](SQL/):

| Script | Purpose |
|---|---|
| `01_create_database.sql` | Create the `SupplyChainDB` database |
| `02_staging_table.sql` | Create a staging table matching the clean CSV |
| `03_bulk_insert.sql` | Bulk-load the clean CSV into staging |
| `04_dimension_tables.sql` | Build dimension tables for the star schema |
| `05_fact_table.sql` | Build `FACT_Orders` |
| `06_rebuild_dim_region.sql` | Split region-level and city-level detail into `DIM_Region` / `DIM_Geography` |
| `07_validation.sql` | Validate loaded data against the Python KPI output |
| `08_kpi_views.sql` | Create reporting views (below) |
| `09_view_verification.sql` | Verify all views exist and return expected shapes |
| `10_fix_dim_shipping_join.sql` | Fix a shipping dimension join issue |

KPI views powering the dashboard:
- `VW_KPI_Summary` — top-level KPI cards (OTD, OTIF, SLA breach, Perfect Order, revenue, profit)
- `VW_KPI_By_ShippingMode` — performance breakdown by shipping mode
- `VW_KPI_By_Region` — regional performance, ranked worst-to-best on OTD
- `VW_SLA_Heatmap` — region × shipping mode cross-analysis with risk categories
- `VW_Monthly_Trend` — month-over-month KPI trends (full years only, 2015–2017)
- `VW_KPI_By_Category` — delivery performance by product category

### 3. Power BI — Dashboard
[`Supply chain- DashBoard.pbix`](Power%20BI%20file/Supply%20chain-%20DashBoard.pbix) connects to the SQL views above across four pages:

| Page | Screenshot |
|---|---|
| Executive Summary — KPI cards, revenue/profit, OTD & SLA breach by shipping mode | [`1_summary.png`](Screenshots/1_summary.png) |
| Regional Analysis — OTD rate by region, filterable by year and shipping mode | [`2_regional_analysis.png`](Screenshots/2_regional_analysis.png) |
| SLA Heatmap — region × shipping mode breach-rate matrix | [`3_sla.png`](Screenshots/3_sla.png) |
| Trend Analysis — monthly/quarterly KPI trends | [`4_trend_analysis.png`](Screenshots/4_trend_analysis.png) |

## Tech Stack

- **Python**: pandas, numpy, matplotlib, seaborn, scipy
- **SQL Server**: T-SQL, star schema (fact + dimension tables), views
- **Power BI**: DirectQuery/Import from SQL Server views

## Reproducing This Project

1. Run the notebook in [`Python/`](Python/) against the raw DataCo CSV to produce a clean CSV.
2. Run the scripts in [`SQL/`](SQL/) in numeric order against a SQL Server instance to build the database, star schema, and KPI views.
3. Open [`Power BI file/Supply chain- DashBoard.pbix`](Power%20BI%20file/Supply%20chain-%20DashBoard.pbix) in Power BI Desktop and point it at your SQL Server instance to refresh.

## Dataset

[DataCo Supply Chain Dataset](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis) — 180,519 order line items, 2015–2018 (2018 is a partial year and is excluded from trend analysis).
