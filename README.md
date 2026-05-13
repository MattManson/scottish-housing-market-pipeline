# scottish-housing-market-pipeline
analysing the housing market in scotland because my house won't sell

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
| 09 | Scottish Postcode Lookup | Complete | Complete | 161,268 active postcodes, includes easting/northing for spatial joins |
| 10 | Scottish GDP | Complete | Complete | 436 rows, 1998 Q1 to 2025 Q4, Total GVA, four measure types |
| 11 | House Building Starts/Completions | Complete | Complete | 32,142 rows, 1996 Q3 to 2024 Q4, council area and country level |
| 12 | Google Trends | Complete | Complete | 261 weeks, May 2021 to May 2026, 5 keywords, GB-SCT |
| 13 | Edinburgh Festival Dates | Complete | Complete | 46 rows, Jul/Aug 2004 to 2026, festival days per month |
| 14 | Met Office Historic Weather | Dropped | Dropped | Scottish weather is a constant not a variable |
| 15 | School Catchments | Complete | Complete | 164,029 rows, postcode to secondary school lookup, 99.5% match rate |
| 16 | EPC Ratings | Dropped | Dropped | Too coarse without property-level transaction data |
| 17 | Recorded Crime | Complete | Complete | 7,656 rows, 1996 to 2024, council area, group aggregates |
| 18 | Transport Accessibility | Complete | Complete | 161,268 rows, bus stops within 500m and rail/ferry within 1000m per postcode |

## Gold Layer (Planned)

All gold tables to be built next session. Join keys are `year_month` for monthly tables and `council_area_code` / `postcode` for geographic joins.

- Nominal vs real house prices (HPI + CPIH deflation)
- Affordability ratio: mean house price / (AWE x 52)
- Base rate vs transaction volume correlation
- Mortgage approvals vs transaction volume (leading indicator lag analysis)
- Claimant count vs transaction volume by local authority
- Edinburgh vs Scotland price comparison
- Festival month effect on Edinburgh transaction volumes
- LBTT price band distribution over time
- Housebuilding starts vs completions lag
- Google Trends leading indicator vs HPI
- School catchment premium on Edinburgh prices
- Transit accessibility score vs average house price
- Crime rate vs house price by council area

## Known Gotchas
- Unity Catalog must be created with explicit MANAGED LOCATION pointing at ADLS Gen2
- BoE endpoints require User-Agent header to bypass Akamai block
- `date` is a reserved word -- use `report_date`
- `date_format` requires `.cast("timestamp")` on date columns
- Two-digit year ambiguity in mmm-yy format -- use Python UDF instead of Spark to_date
- Delta schema conflict if filtered-on column passed through to Spark -- filter in pandas first
- UDF definition can reset active catalog -- use fully qualified table names after any UDF
- NOMIS API has a 25k row cap -- split large date ranges into multiple requests
- Delta rejects column names with spaces or special characters -- sanitise in pandas first using clean_col_name() pattern
- pytrends requires urllib3==1.26.18 to avoid method_whitelist error
- statistics.gov.scot CSV pattern: `https://statistics.gov.scot/downloads/cube-table?uri=http://statistics.gov.scot/data/{dataset-name}`
- NaPTAN Scotland filter: ATCOCode starts with '6'
- After %pip install and restartPython() all variables are wiped -- rerun all cells in order

## Medallion Layers

- **Bronze:** Raw ingest from each source, minimal transformation, column names sanitised for Delta compatibility
- **Silver:** Cleaned, typed, normalised, joined to a common year_month key (YYYY-MM string). Spatial tables use easting/northing in EPSG:27700
- **Gold:** Aggregated analytical tables for dashboard consumption

## Setup Notes

- Unity Catalog metastore must be configured with ADLS Gen2 as managed storage location before creating schemas
- Catalog storage root: `abfss://data@scottishhousingdl.dfs.core.windows.net/`
- Four schemas required: bronze, silver, gold, config
- School catchment shapefiles stored at: `abfss://data@scottishhousingdl.dfs.core.windows.net/raw/school_catchments/`

## Known Limitations

- RoS median price data unavailable due to Cloudflare protection -- mean price from UK HPI used throughout
- BoE mortgage approvals and ONS AWE are GB-wide figures, not Scotland-specific -- noted on dashboard
- LBTT data only available from April 2015 (when it replaced SDLT in Scotland)
- Google Trends values are 0-100 relative index within a single request -- not comparable to external benchmarks
- School catchment boundaries current as of January 2024 -- historical catchment changes not captured
- NaPTAN transport accessibility is a point-in-time snapshot -- does not capture historical service changes
```

---