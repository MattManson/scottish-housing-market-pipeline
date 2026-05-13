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

## Source Status

| # | Source | Bronze | Silver | Notes |
|---|--------|--------|--------|-------|
| 01 | UK HPI (Land Registry) | Complete | Complete | 9,207 Scottish rows, Jan 1968 to Feb 2026 |
| 02 | RoS House Price Stats | Skipped | Skipped | Cloudflare blocked, documented in notebook |
| 03 | BoE Base Rate | Complete | Complete | 268 rows, Jan 2004 to Apr 2026 |
| 04 | BoE Mortgage Approvals | Complete | Complete | 267 rows, Jan 2004 to Mar 2026, UK-wide |
| 05 | ONS CPIH | Complete | Complete | 1,828 rows, 4 components, Jan 1988 to Jan 2026 |
| 06 | ONS AWE | Complete | Complete | 314 rows, Jan 2000 to Feb 2026, GB-wide |
| 07 | LBTT Statistics | Complete | Complete | 132 rows each, Apr 2015 to Mar 2026, two tables |
| 08 | DWP Claimant Count | Complete | Complete | 3,840 rows, 32 councils, Apr 2016 to Mar 2026 |
| 09 | Scottish Postcode Lookup | Complete | Complete | 161,268 active postcodes, one-time load |

## Planned Additional Sources

| # | Source | Priority | Notes |
|---|--------|----------|-------|
| 10 | Scottish GDP (Scottish Government) | High | Quarterly, Scotland-specific economic indicator |
| 11 | House Building Starts/Completions (Scottish Government) | High | Supply-side context |
| 12 | Google Trends | High | Search interest as leading indicator, pytrends |
| 13 | Edinburgh Festival Dates | High | Boolean flag, August transaction volume dip |
| 14 | Met Office Historic Weather | Medium | Regional, correlates with transaction volumes |
| 15 | School Performance (Scotxed) | Medium | Catchment effect on Edinburgh prices |
| 16 | EPC Ratings | High | Property-level, joins to postcode lookup, highest payoff |

## Known Gotchas
- Unity Catalog must be created with explicit MANAGED LOCATION pointing at ADLS Gen2
- BoE endpoints require User-Agent header to bypass Akamai block
- `date` is a reserved word -- use `report_date`
- `date_format` requires `.cast("timestamp")` on date columns
- Two-digit year ambiguity in mmm-yy format -- use Python UDF instead of Spark to_date
- Delta schema conflict if filtered-on column passed through to Spark -- filter in pandas first
- UDF definition can reset active catalog -- use fully qualified table names after any UDF
- NOMIS API has a 25k row cap -- split large date ranges into multiple requests

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
