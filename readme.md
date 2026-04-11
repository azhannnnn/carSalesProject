# Scalable Car Sales Analytics Platform
### Azure · ADF · Databricks · PySpark · Delta Lake · Unity Catalog · Power BI

A production-grade cloud analytics platform built on **Azure Medallion Architecture**, ingesting and transforming 5M+ car sales records into actionable business intelligence dashboards.

---

## Architecture Overview

```
Raw Data Sources (APIs / CSV)
        │
        ▼
┌─────────────────────────────────────────────────────┐
│              Azure Data Factory (ADF)               │
│         Orchestration · Scheduling · Ingestion      │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│         Azure Data Lake Storage Gen2 (ADLS)         │
│                                                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │   BRONZE    │→ │   SILVER    │→ │    GOLD     │ │
│  │  Raw data   │  │  Cleaned &  │  │  Aggregated │ │
│  │  as-is      │  │  validated  │  │  & modeled  │ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│            Azure Databricks + PySpark               │
│     Transformation · Star Schema · SCD Type 2       │
│              Delta Lake · Unity Catalog             │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│                   Power BI                          │
│         Executive Dashboards · Sales Insights       │
└─────────────────────────────────────────────────────┘
```

---

## What This Project Does

| Layer | Description |
|---|---|
| Ingestion | ADF pipelines pull data from API sources and land raw files in ADLS Gen2 Bronze layer |
| Transformation | PySpark notebooks on Databricks clean, validate, and enrich data through Silver layer |
| Modeling | Star Schema with Fact and Dimension tables, SCD Type 2 for full historical tracking |
| Governance | Unity Catalog enforces role-based data access and column-level security |
| Reporting | Power BI dashboards connect directly to Gold layer for executive-level insights |

---

## Tech Stack

| Category | Tools |
|---|---|
| Cloud Platform | Microsoft Azure |
| Orchestration | Azure Data Factory (ADF) |
| Storage | Azure Data Lake Storage Gen2 (ADLS Gen2) |
| Processing | Azure Databricks, Apache Spark, PySpark |
| Table Format | Delta Lake |
| Data Modeling | Star Schema, SCD Type 2 |
| Governance | Unity Catalog |
| Visualization | Power BI |
| Language | Python, PySpark, SQL |

---

## Data Model

```
                    ┌──────────────────┐
                    │   FACT_SALES     │
                    │──────────────────│
                    │ sale_id (PK)     │
              ┌────▶│ car_id (FK)      │◀────┐
              │     │ dealer_id (FK)   │     │
              │     │ date_id (FK)     │     │
              │     │ customer_id (FK) │     │
              │     │ sale_price       │     │
              │     │ quantity         │     │
              │     │ discount_amount  │     │
              │     └──────────────────┘     │
              │                             │
┌─────────────┴──┐              ┌───────────┴────┐
│   DIM_CAR      │              │  DIM_DEALER     │
│────────────────│              │─────────────────│
│ car_id (PK)    │              │ dealer_id (PK)  │
│ make           │              │ dealer_name     │
│ model          │              │ region          │
│ year           │              │ city            │
│ category       │              │ state           │
│ engine_type    │              │ country         │
│ effective_date │  SCD Type 2  │ effective_date  │
│ end_date       │◀────────────▶│ end_date        │
│ is_current     │              │ is_current      │
└────────────────┘              └─────────────────┘

┌──────────────────┐            ┌──────────────────┐
│  DIM_DATE        │            │  DIM_CUSTOMER    │
│──────────────────│            │──────────────────│
│ date_id (PK)     │            │ customer_id (PK) │
│ full_date        │            │ customer_name    │
│ day              │            │ age_group        │
│ month            │            │ region           │
│ quarter          │            │ segment          │
│ year             │            │ effective_date   │
│ is_weekend       │            │ end_date         │
│ fiscal_quarter   │            │ is_current       │
└──────────────────┘            └──────────────────┘
```

---

## Medallion Architecture — Layer by Layer

### Bronze Layer (Raw)
- Raw data landed exactly as received from source
- No transformations applied
- Full audit trail preserved
- Partitioned by ingestion date

### Silver Layer (Cleaned)
- Null handling and schema enforcement
- Duplicate removal and deduplication logic
- Data type standardization
- Business rule validations applied
- ~35% reduction in data quality issues vs raw layer

### Gold Layer (Aggregated)
- Star Schema dimensional model
- Pre-aggregated metrics for fast BI queries
- SCD Type 2 implemented for dealer and customer dimensions
- Optimized Delta tables with Z-ordering for query performance

---

## Key Features

- **5M+ records** processed end-to-end through the pipeline
- **SCD Type 2** historical tracking on dealer and customer dimensions — full history preserved, never overwritten
- **Incremental loads** — ADF pipelines detect and load only changed records on each run
- **Data quality checks** at every layer — schema validation, null checks, referential integrity
- **Unity Catalog governance** — column-level security, row filters, full data lineage
- **Power BI live connection** to Gold layer — dashboards refresh automatically on pipeline completion

---

## Pipeline Runs — ADF Orchestration

```
Daily Schedule (11:00 PM IST)
        │
        ├── 1. Extract from API source → Bronze ADLS
        │
        ├── 2. Trigger Databricks Bronze → Silver notebook
        │        └── Schema validation
        │        └── Null handling
        │        └── Deduplication
        │
        ├── 3. Trigger Databricks Silver → Gold notebook
        │        └── Star Schema load
        │        └── SCD Type 2 merge
        │        └── Delta OPTIMIZE + VACUUM
        │
        └── 4. Notify on success / failure via ADF alerts
```

---

## Power BI Dashboard Highlights

- **Revenue by Region** — geo map with drill-through to dealer level
- **Monthly Sales Trend** — line chart with YoY comparison
- **Top 10 Car Models** — ranked by revenue and units sold
- **Dealer Performance Scorecard** — KPIs with conditional formatting
- **Customer Segment Analysis** — age group and regional breakdown

---

## Project Structure

```
car-sales-analytics/
│
├── adf/
│   ├── pipelines/          # ADF pipeline JSON exports
│   └── linked_services/    # Connection configs
│
├── databricks/
│   ├── bronze_to_silver.py # Bronze → Silver transformation
│   ├── silver_to_gold.py   # Silver → Gold modeling
│   ├── scd_type2_merge.py  # SCD Type 2 merge logic
│   └── data_quality.py     # Validation framework
│
├── sql/
│   ├── create_tables.sql   # Gold layer table DDLs
│   └── quality_checks.sql  # Data quality queries
│
├── powerbi/
│   └── car_sales_report.pbix
│
└── README.md
```

---

## How to Run

### Prerequisites
- Azure subscription with Databricks and ADLS Gen2 provisioned
- Azure Data Factory instance
- Unity Catalog metastore configured

### Steps

```bash
# 1. Clone the repo
git clone https://github.com/azhannnnn/car-sales-analytics.git
cd car-sales-analytics

# 2. Configure ADLS Gen2 connection in Databricks
# Update the storage account name and access key in:
# databricks/bronze_to_silver.py → line 12

# 3. Import ADF pipelines
# In Azure Data Factory → Manage → Import from adf/pipelines/

# 4. Run Databricks notebooks in order:
#    bronze_to_silver.py → silver_to_gold.py

# 5. Connect Power BI to Gold layer
# Open powerbi/car_sales_report.pbix
# Update data source to your ADLS Gen2 endpoint
```

---

## Results

| Metric | Value |
|---|---|
| Total records processed | 5M+ |
| Pipeline execution time | ~18 minutes end-to-end |
| Data quality improvement | ~35% defect reduction vs raw |
| Dashboard refresh latency | < 5 minutes after pipeline completion |
| Historical tracking | Full SCD Type 2 — no data loss |

---

## About the Author

**Azhan Khan** — Data Engineer specializing in Azure data platforms, ETL pipelines, and ML data infrastructure.

- Previously built ML training data pipelines for **NVIDIA's autonomous driving program** at Zensar Technologies — processing 100K+ multi-modal records daily
- Databricks Certified · Oracle Multicloud Architect · LeetCode World Rank 321

Connect: [LinkedIn](https://linkedin.com/in/azhankhan22) · [GitHub](https://github.com/azhannnnn)

---

*Built with Azure Data Factory · Azure Databricks · Delta Lake · PySpark · Unity Catalog · Power BI*
