# scottish-housing-market-pipeline
analysing the hosuing market in scotland because my house won't sell


# Scottish Housing Market Pipeline

A Databricks portfolio project analysing the Scottish housing market, correlating house prices with economic indicators. Dual purpose: a genuinely useful personal tool (house currently on market) and a CV portfolio piece demonstrating the modern data stack on Databricks.

## Architecture

- **Platform:** Azure Databricks (North Europe)
- **Storage:** Azure Data Lake Storage Gen2
- **Format:** Delta Lake (bronze/silver/gold medallion architecture)
- **Orchestration:** Databricks Workflows
- **Governance:** Unity Catalog
- **Language:** Python / PySpark

## Data Sources

| # | Source | Format | Status | Notes |
|---|--------|--------|--------|-------|
| 01 | UK HPI Full File (Land Registry) | CSV | Complete | Mean price, volume, property type for Scotland + 32 local authorities |
| 02 | RoS House Price Statistics | XLSX | Skipped | Cloudflare blocks all programmatic access. Would have provided median price by local authority. Manual download available at ros.gov.uk |
| 03 | BoE Base Rate | CSV via API | Pending | |
| 04 | BoE Mortgage Approvals | CSV via API | Pending | |
| 05 | ONS CPIH | CSV via version discovery | Pending | |
| 06 | ONS Average Weekly Earnings | CSV | Pending | |
| 07 | LBTT Statistics (Revenue Scotland) | ODS | Pending | |
| 08 | DWP Claimant Count (NOMIS) | CSV via API | Pending | |
| 09 | Scottish Postcode Lookup (NRS) | ZIP/CSV | Pending | |

## Medallion Layers

- **Bronze:** Raw ingest from each source, minimal transformation, metadata columns added (source_url, ingested_at)
- **Silver:** Cleaned, typed, normalised, joined to a common year_month key (YYYY-MM string)
- **Gold:** Aggregated analytical tables for dashboard consumption

## Setup Notes

- Unity Catalog metastore must be configured with ADLS Gen2 as managed storage location before creating schemas
- Catalog storage root: `abfss://data@scottishhousingdl.dfs.core.windows.net/`
- Four schemas required: bronze, silver, gold, config

## Known Limitations

- RoS median price data unavailable due to Cloudflare protection on ros.gov.uk -- mean price from UK HPI used throughout instead
- BoE mortgage approvals and ONS AWE are GB-wide figures, not Scotland-specific -- noted on dashboard
- LBTT data only available from April 2015 (when it replaced SDLT in Scotland)
```

---
