scottish-housing-market-pipeline
analysing the housing market in scotland because my house won't sell
Scottish Housing Market Pipeline
A Databricks portfolio project analysing the Scottish housing market, correlating house prices with economic indicators. Dual purpose: a genuinely useful personal tool (house currently on market) and a CV portfolio piece demonstrating the modern data stack on Databricks.
Architecture
Platform: Azure Databricks (North Europe)
Storage: Azure Data Lake Storage Gen2
Format: Delta Lake (bronze/silver/gold medallion architecture)
Orchestration: Databricks Workflows (planned)
Governance: Unity Catalog
Language: Python / PySpark
Source Status
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
Gold Layer
All 14 gold tables complete. Each notebook writes to Delta and exports a CSV to ADLS for portability.
CSV export root: abfss://data@scottishhousingdl.dfs.core.windows.net/exports/
| # | Gold Table | Sources | Grain | Rows | Key Finding |
|---|-----------|---------|-------|------|-------------|
| 01 | nominal_vs_real_house_prices | uk_hpi + cpih | Monthly, all areas | 8,937 | Since 2015: 43% nominal growth, only 4.8% real growth |
| 02 | affordability_ratio | uk_hpi + ons_awe | Monthly, Scotland | 314 | Ratio risen from 3.1 years in 2000 to 4.9 years in 2026 |
| 03 | base_rate_vs_transaction_volume | boe_base_rate + uk_hpi | Monthly, all areas | 8,778 | Rate rises in 2022-23 visible in transaction volume data |
| 04 | mortgage_approvals_vs_transaction_volume | boe_mortgage_approvals + uk_hpi | Monthly, Scotland | 266 | 3-month lagged approvals included as leading indicator |
| 05 | claimant_count_vs_transaction_volume | dwp_claimant_count + uk_hpi | Monthly, by council | 3,332 | Council-level labour market proxy joined to price data |
| 06 | edinburgh_vs_scotland_price_comparison | uk_hpi | Monthly | 266 | Edinburgh consistently 55-65% above Scotland average |
| 07 | festival_month_effect | uk_hpi + lbtt_table_1 + festival_dates | Monthly, Edinburgh | 266 | Festival months average 18% higher transaction volumes |
| 08 | lbtt_price_band_distribution | lbtt_table_2 | Monthly, Scotland | 132 | £325k-750k band grown from 3.2% to 17.3% of all transactions since 2015 |
| 09 | housebuilding_starts_vs_completions_lag | housebuilding | Quarterly, all areas | 13,332 | Starts down ~40% since late 1990s; completions outpacing starts |
| 10 | google_trends_vs_hpi | google_trends + uk_hpi | Monthly, Scotland + Edinburgh | 116 | Search interest has halved since 2021 covid boom |
| 11 | catchment_characterisation | school_catchments + postcode_lookup + uk_hpi | By catchment | 370 | Deprivation and urban/rural profile per secondary school catchment |
| 12 | transit_accessibility_vs_price | transport_accessibility + uk_hpi | By council area | 32 | Transport access does not strongly predict council-level prices |
| 13 | crime_rate_vs_house_price | crime + uk_hpi | Annual, by council | 672 | Edinburgh crimes fell 41% since 2004 while prices doubled |
| 14 | gdp_vs_house_price | scottish_gdp + uk_hpi | Quarterly, Scotland | 112 | House prices have grown 285 percentage points more than GDP since 1998 |
Limitations on gold_11 and gold_12
Property-level transaction data from Registers of Scotland is unavailable (Cloudflare protected). As a result:
catchment_characterisation profiles catchments by deprivation and urban/rural character but cannot compute per-catchment price premiums
transit_accessibility_vs_price joins transport metrics to council-level HPI rather than postcode-level prices
Both tables are clearly documented as characterisation tables rather than price premium analyses.
Key Analytical Findings
Real vs nominal prices
Scottish house prices have grown ~429% since 1998. Scottish GDP has grown ~43% over the same period. The gap -- 285 percentage points -- is structural and shows no sign of closing. It originated in the 2002-2007 credit boom and has never reversed.
What actually drives Scottish house prices
Credit conditions and interest rates, not GDP. The 2008 financial crisis is the only period where GDP and house prices moved in lockstep. Covid produced a -21% GDP quarter while house prices rose 1.4% simultaneously. Housing follows its own logic.
Edinburgh specifically
Edinburgh real prices peaked in mid-2021 and have declined since. The Edinburgh premium over the Scotland average has been stable at 55-65% in percentage terms for two decades but has doubled in cash terms from ~£45k to ~£108k.
Supply
Housebuilding starts have fallen ~40% since the late 1990s and the pipeline is shrinking. A supply constraint story -- but a 3-5 year effect, not a 12-month one.
Portability
The Databricks trial has a finite lifespan. The gold layer has been designed for easy migration:
All gold tables use simple types (date, string, double, long, boolean) -- no Databricks-specific features
CSV exports written to ADLS alongside every Delta table
CSVs load into Postgres via \copy with no conversion required
Notebooks exported as .ipynb for GitHub and re-use on other platforms
Likely hosting path: Streamlit dashboard pointing at a Postgres instance seeded from the CSV exports.
Known Snags
| Snag | Detail |
|------|--------|
| silver.uk_hpi column naming | Renamed to snake_case mid-session (Date to report_date, RegionName to region_name etc). Confirm bronze notebook is consistent. |
| festival_days_in_month 2020-08 | Shows 31 instead of 0 for cancelled festival year. Minor silver data quality issue. |
| google_trends silver grain | Weekly not monthly -- must aggregate in gold before joining to HPI. Handled in gold_10. |
| gold_01 column name | Uses nominal_average_price not average_price -- inconsistent with other gold tables. Note when querying. |
Known Gotchas
Unity Catalog must be created with explicit MANAGED LOCATION pointing at ADLS Gen2
BoE endpoints require User-Agent header to bypass Akamai block
date is a reserved word -- use report_date
date_format requires .cast("timestamp") on date columns
Two-digit year ambiguity in mmm-yy format -- use Python UDF instead of Spark to_date
Delta schema conflict if filtered-on column passed through to Spark -- filter in pandas first
UDF definition can reset active catalog -- use fully qualified three-part table names after any UDF
NOMIS API has a 25k row cap -- split large date ranges into multiple requests
Delta rejects column names with spaces or special characters -- sanitise in pandas first using clean_col_name()
pytrends requires urllib3==1.26.18 to avoid method_whitelist error
statistics.gov.scot CSV pattern: https://statistics.gov.scot/downloads/cube-table?uri=http://statistics.gov.scot/data/{dataset-name}
NaPTAN Scotland filter: ATCOCode starts with '6'
After %pip install and restartPython() all variables are wiped -- rerun all cells in order
Division by zero in Spark with / operator -- use F.when(col == 0, None).otherwise(col_a / col_b)
Window function requires explicit import -- from pyspark.sql.window import Window -- add to cell where Window is used if import scope fails
Google Trends silver is weekly grain -- filter is_partial == False and aggregate to monthly before joining to HPI
Medallion Layers
Bronze: Raw ingest from each source, minimal transformation, column names sanitised for Delta compatibility. One notebook per source.
Silver: Cleaned, typed, normalised. Common report_date (DateType) and year_month (YYYY-MM string) columns for joining. All column names snake_case. Spatial tables use easting/northing in EPSG:27700. One notebook per source.
Gold: Analytical tables joining across silver sources. Aggregated to appropriate grain. Each notebook writes Delta table and CSV export. 14 tables complete.
Setup Notes
Unity Catalog metastore must be configured with ADLS Gen2 as managed storage location before creating schemas
Catalog storage root: abfss://data@scottishhousingdl.dfs.core.windows.net/
Four schemas required: bronze, silver, gold, config
School catchment shapefiles stored at: abfss://data@scottishhousingdl.dfs.core.windows.net/raw/school_catchments/
Known Limitations
RoS median price data unavailable due to Cloudflare protection -- mean price from UK HPI used throughout
BoE mortgage approvals and ONS AWE are GB-wide figures, not Scotland-specific
LBTT data only available from April 2015 (when it replaced SDLT in Scotland)
Google Trends values are 0-100 relative index within a single request -- not comparable to external benchmarks
School catchment boundaries current as of January 2024 -- historical catchment changes not captured
NaPTAN transport accessibility is a point-in-time snapshot -- does not capture historical service changes
Scottish GDP covers onshore economy only -- excludes North Sea oil and gas extraction
Crime data is fiscal year granularity -- joined to calendar year HPI as an approximation
gold_11 and gold_12 cannot compute property-level price premiums without RoS transaction data
Next Steps
Export all notebooks as .ipynb before Databricks trial ends
Fix silver.uk_hpi snag -- confirm snake_case column rename persisted correctly
Fix festival_days_in_month for 2020-08 (should be 0, not 31)
Standardise gold column naming (nominal_average_price vs average_price)
Plan and build dashboard layer (Streamlit + Postgres most likely path)
Decide hosting approach before trial ends
