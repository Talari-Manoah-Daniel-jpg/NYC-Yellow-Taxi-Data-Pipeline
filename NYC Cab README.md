# NYC Yellow Taxi Data Pipeline

End-to-end batch data pipeline for NYC Yellow Taxi trip data, built on Databricks using PySpark, Delta Lake, and a medallion (Bronze / Silver / Gold) architecture. Orchestrated as a multi-task Databricks Job and visualized in Power BI.

## Architecture

Raw data → **Bronze** (raw ingestion) → **Silver** (cleaned + quarantined) → **Gold** (business aggregates) → **Power BI dashboard**

- **Bronze** — Ingests the monthly NYC TLC Yellow Taxi trip data (parquet) and taxi zone lookup (CSV) from the official TLC source, landing them as Delta tables with ingestion metadata (`_ingested_at`, `_source_file`). Files are staged through a Unity Catalog Volume, since Databricks Free Edition's serverless compute blocks direct local-filesystem access from Spark. Reloads are idempotent — re-running a load for the same month deletes the prior copy first, rather than duplicating it.
- **Silver** — Applies explicit data quality rules. Rows with an invalid fare, distance, duration, or an unmapped pickup/dropoff zone are hard-rejected into a separate `silver_yellow_taxi_rejected` table with a reason code per row. Unreliable `passenger_count` values are soft-corrected (nulled) rather than discarding the whole trip. Clean trips are enriched with pickup/dropoff borough and zone names.
- **Gold** — Five pre-aggregated tables, each answering one specific business question: daily trend, hourly demand patterns, busiest zones, payment mix, and borough-to-borough flow.
- **Orchestration** — A 3-task Databricks Job (Bronze → Silver → Gold) with dependency-based execution, parameterized by month (`year_month`) to support backfilling additional months.
- **Visualization** — Power BI dashboard built from the Gold tables (KPI cards, daily trend line, hour/day heatmap, top zones, payment mix, borough flow matrix).

## Results (May 2026 data)

- 4,090,836 raw trip records processed
- 4.4% rejection rate (179,015 rows quarantined with an explicit reason each)
- 5 Gold aggregate tables feeding a 6-visual Power BI dashboard

## Notebooks

| Notebook | Purpose |
|---|---|
| `01_bronze_ingestion.py` | Downloads and lands raw trip + zone data |
| `02_silver_cleaning.py` | Cleans, quarantines, and enriches trip data |
| `03_gold_aggregation.py` | Builds the 5 business-ready aggregate tables |

## Documentation

See the included explainer document for a full plain-English, line-by-line walkthrough of every piece of code in this pipeline.

## Dashboard

See the `screenshots` folder for the final Power BI dashboard.

## Tools

Databricks, PySpark, Delta Lake, Unity Catalog Volumes, Databricks Workflows, Power BI Desktop.
