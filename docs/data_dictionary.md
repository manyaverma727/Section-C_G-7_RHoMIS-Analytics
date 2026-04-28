# Data Dictionary

**Dataset:** RHoMIS — Rural Household Multi-Indicator Survey
**Source:** Harvard Dataverse — DOI: [10.7910/DVN/WS38SA](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi%3A10.7910%2FDVN%2FWS38SA)
**Coverage:** 54,873 households across 35 countries, 2015–2023
**Pipeline:** Raw → `scripts/etl_pipeline.py` → `rhomis_cleaned.csv` → `notebooks/05_final_load_prep.ipynb` → `rhomis_tableau_ready.csv`

---

## Files

| File | Location | Rows | Columns | Description |
|---|---|---|---|---|
| `raw.csv.gz` | `data/raw/` | 54,873 | 1,599 | Original survey data, never edited |
| `rhomis_cleaned.csv` | `data/processed/` | 54,873 | 77 | Structurally cleaned output from ETL pipeline |
| `rhomis_cleaned.csv.gz` | `data/processed/` | 54,873 | 77 | Compressed copy of cleaned data |
| `rhomis_cleaning_summary.json` | `data/processed/` | — | — | ETL run metadata, output schema, missing value counts |
| `rhomis_tableau_ready.csv` | `data/processed/` | 54,873 | 46 | Final analysis-ready export with derived KPI fields |

---

## Analysis Dataset — rhomis_tableau_ready.csv (46 columns)

### Identity and Geography

| Column | Type | Missing | Range / Values | Description |
|---|---|---|---|---|
| `id_unique` | string | 0% | 54,873 unique | Unique household identifier. Format: `{country}_{project}_{year}_{n}`. Primary key. |
| `country` | string | 0% | 35 values | Country name in lowercase. Aliased to display names in Tableau. |
| `iso_country_code` | string | 0% | 35 values | ISO 3166-1 alpha-2 country code. Used for choropleth map geographic role. |
| `year` | integer | 0% | 2015–2023 | Survey year. Distribution is uneven (bulk in 2018–2022). Do not use for time-series trending. |
| `region` | string | 10.8% | 196 values | Sub-national region name. Not standardised across countries. |
| `gps_lat_rounded` | float | 0.7% | -29 to 51 | GPS latitude rounded to 1 decimal place for privacy. |
| `gps_lon_rounded` | float | 0.7% | -90 to 120 | GPS longitude rounded to 1 decimal place for privacy. |

---

### Food Security

| Column | Type | Missing | Range / Values | Description |
|---|---|---|---|---|
| `foodshortagetime` | string | 13.6% | `y`, `n` | Self-reported binary: did the household experience a food shortage period in the past year. |
| `food_shortage_flag` | float | 13.6% | 0 or 1 | Numeric encoding of `foodshortagetime`. 1 = food shortage, 0 = none. Primary outcome variable. Global rate: 67.7%. |
| `food_shortage_status` | string | 13.6% | `Food Shortage`, `No Food Shortage` | Display label derived from `food_shortage_flag`. Used for Tableau categorical filters. |
| `nr_months_food_shortage` | float | 48.0% | 1–12 | Number of months the household experienced food shortage. Only populated where `food_shortage_flag` = 1. Median: 3 months. |
| `fies_yes_count` | float | 54.3% | 0–8 | Count of affirmative responses to the 8 FIES (Food Insecurity Experience Scale) questions. Higher = more insecure. Available for 22 of 35 countries. |
| `hfias_status_indicator` | string | 0% | `food_secure`, `mildly_fi`, `moderately_fi`, `severely_fi` | Household Food Insecurity Access Scale classification. 100% coverage but many countries show 100% food_secure, indicating the instrument was not deployed properly in those waves. |
| `hfias_frequency_score` | float | 87.5% | 0–45 | Numeric frequency score from HFIAS questions. 87.5% missing — do not use for country-level comparisons. Use `hfias_status_indicator` instead. |
| `hdds_good_season` | float | 44.0% | 0–10 | Household Dietary Diversity Score during the good agricultural season. |
| `hdds_bad_season` | float | 38.0% | 0–10 | Household Dietary Diversity Score during the lean season. |
| `hdds_last_month` | float | 68.6% | 0–10 | Household Dietary Diversity Score in the month preceding the survey. 68.6% missing — avoid in primary analysis. |

---

### Demographics

| Column | Type | Missing | Range / Values | Description |
|---|---|---|---|---|
| `household_size_derived` | integer | 0% | 0–369 | Total household members summed from 8 age-band columns. Zero missing values. Preferred over `count_people` (49% missing in cleaned data). |
| `hh_size_mae` | float | 0% | 0–289 | Household size in Male Adult Equivalents — a welfare economics unit that weights members by age and sex for income normalisation. |
| `respondentsex` | string | 11.2% | `m`, `f` | Sex of the survey respondent. |
| `female_respondent_flag` | float | 11.2% | 0 or 1 | 1 if respondent is female. Derived from `respondentsex`. |
| `education_level` | string | 17.1% | 71 values | Respondent's highest education level. Not standardised across countries. |
| `low_education_flag` | float | 17.1% | 0 or 1 | 1 if respondent has no formal education or primary only. Derived from `education_level`. |
| `respondent_is_head` | string | 14.0% | `y`, `n` | Whether the survey respondent is the household head. |

---

### Farm and Land

| Column | Type | Missing | Range / Values | Description |
|---|---|---|---|---|
| `land_cultivated_ha` | float | 6.1% | 0–30,000 | Cultivated land area in hectares. Converted from local units using a 28-unit conversion table in the ETL. Unknown units produce NaN — no guessing. Max of 30,000 ha (Nigeria, unit=hectare) is a legitimate large farm. |
| `land_tenure` | string | 9.6% | 217 values | Type of land tenure arrangement (owned, rented, communal, etc.). Not standardised. |
| `land_ownership` | string | 36.1% | 76 values | Who holds ownership rights to the land. |
| `land_irrigated` | string | 8.6% | `y`, `n` | Whether the farm uses irrigation. |
| `irrigated_flag` | float | 8.6% | 0 or 1 | Numeric encoding of `land_irrigated`. Irrigated households show 16pp lower food shortage rates than rainfed (55.2% vs 71.5%). |
| `farm_labour` | string | 17.7% | 56 values | Primary labour source for farming. |
| `crop_diversity_count` | float | 0% | 0–10 | Number of distinct crop types grown by the household. Median: 2. Bulk of households grow 1–5 crops. |
| `crop_harvest_total_kg_observed` | float | 9.0% | 0.3–31 billion | Total crop harvest in kg across all observed crops. Extreme right skew (skewness 223) — never use on a continuous axis. Use log scale or median aggregation only. |
| `land_productivity_kg_per_ha` | float | 12.9% | 0.01–1.8 billion | Total crop harvest divided by cultivated land. Extreme right skew (skewness 190). Use log scale or quartile bins. Median: 600 kg/ha. |
| `productivity_quartile` | string | 12.9% | `Q1 lowest`, `Q2`, `Q3`, `Q4 highest` | Land productivity binned into quartiles based on non-missing values. Q1 households show 83% food shortage rate vs 54% for Q4. |

---

### Income

| Column | Type | Missing | Range / Values | Description |
|---|---|---|---|---|
| `offfarm_incomes_any` | string | 6.5% | `y`, `n` | Whether the household has any off-farm income source. |
| `offfarm_flag` | float | 6.5% | 0 or 1 | Numeric encoding of `offfarm_incomes_any`. 44.1% of households have off-farm income. |
| `total_income_ppp_per_year` | float | 7.1% | -220 to 350 billion | Total annual household income in PPP USD. Extreme right skew (skewness 209). Never use on continuous axis. Median: $650 PPP/year. |
| `income_ppp_per_mae` | float | 11.1% | -32 to 78 billion | Annual income per Male Adult Equivalent in PPP USD. Extreme right skew (skewness 148). Always use MEDIAN for aggregation. Median: $123 PPP/MAE. |
| `country_income_rank_pct` | float | 7.1% | 0–1 | Within-country income percentile rank. Skewness 0.08 — near-perfectly uniform. Safe for box plots and scatter. 0 = lowest income, 1 = highest within that country. |
| `country_income_quintile` | string | 7.1% | `Lowest`, `Low`, `Middle`, `High`, `Highest` | Income quintile based on `country_income_rank_pct`. Quintile boundaries computed within each country, making comparisons meaningful across countries. |
| `farm_value_consumed_ppp_per_year` | float | 6.9% | -340 to 17 billion | Estimated value of farm produce consumed by the household in PPP USD. Extreme skew — use median only. |
| `farm_products_consumed_calories_kcal_per_hh_per_year` | float | 6.9% | -450,000 to 88 trillion | Estimated caloric value of farm produce consumed. Extreme skew (skewness 226) — do not use on any axis. |
| `livestock_tlu` | float | 5.9% | 0–3,000 | Total livestock holdings in Tropical Livestock Units. Median: 0.95 TLU. |

---

### Gender and Resource Control

| Column | Type | Missing | Range / Values | Description |
|---|---|---|---|---|
| `proportion_of_value_controlled_female_adult` | float | 22.5% | 0–1 | Share of total household agricultural value controlled by female adults. Median: 0.5. Skewness 0.21 — safe for charts. |
| `proportion_of_value_controlled_male_adult` | float | 23.6% | 0–1 | Share of total household agricultural value controlled by male adults. |

---

### Vulnerability Segmentation

| Column | Type | Missing | Range / Values | Description |
|---|---|---|---|---|
| `vulnerability_profile_k4` | string | 23.8% | 4 values | k-means cluster assignment (k=4) based on farm structure and food security indicators. 13,075 households unclassified (insufficient data for clustering). |

**Profile descriptions:**

| Profile | Count | Food Shortage Rate | Interpretation |
|---|---|---|---|
| Low-income low-productivity rainfed | 14,330 | 78.7% | Highest-risk segment. Subsistence farming, no irrigation, minimal off-farm income. |
| Large-household extensive low-productivity | 6,860 | 72.3% | Large families on extensive but unproductive land. High food insecurity despite land access. |
| Diversified rainfed better-off | 12,855 | 62.7% | Multiple income sources, better crop diversity. Still majority food insecure. |
| Irrigated higher-productivity | 7,753 | 55.8% | Lowest-risk segment. Irrigation access drives better outcomes across all indicators. |

---

## Cleaned Dataset — rhomis_cleaned.csv (77 columns)

The 77-column cleaned dataset is the intermediate output of the ETL pipeline. It contains the structural foundation before analysis-phase derivation. Key additions over the raw data:

| Derived Column | Source Columns | Logic |
|---|---|---|
| `household_size_derived` | `children_under_4` through `femalesover50` (8 cols) | Sum of all age-band columns. 0% missing — used in place of `count_people`. |
| `land_cultivated_ha` | `landcultivated`, `unitland` | Unit conversion to hectares using 28 local unit mappings. Unmappable units → NaN. |

**Key cleaning decisions:**
- No imputation applied to any field. Missing values remain NaN throughout.
- `count_people` retained as-is despite 49.4% missing. `household_size_derived` is the reliable alternative.
- `age_malehead` and `age_femalehead` corrected for birth-year contamination using row-level survey year.
- HFIAS (hfias_1–9) and FIES (fies_1–8) retained as validated categorical strings, not scored.
- `offfarm_income_proportion` treated as categorical — not converted to numeric.
- Country names standardised to lowercase. Misspelling `'philipines'` corrected to `'philippines'`.

---

## Data Limitations

| Limitation | Affected Columns | Impact |
|---|---|---|
| Extreme right skew | `income_ppp_per_mae`, `land_productivity_kg_per_ha`, `crop_harvest_total_kg_observed`, `total_income_ppp_per_year` | Cannot use on continuous chart axes. Use median, log scale, or bins. |
| HFIAS not deployed everywhere | `hfias_status_indicator`, `hfias_frequency_score` | Many countries show 100% food_secure — instrument artefact, not genuine outcome. Use `hfias_status_indicator` with caution. Cross-reference with `food_shortage_flag`. |
| FIES coverage partial | `fies_yes_count` | Only 22 of 35 countries. All FIES charts must note limited coverage. |
| Uneven year distribution | `year` | 2018 has 12,375 rows, 2015 has 1,052. Do not use for time-series trend analysis. |
| HDDS last month sparse | `hdds_last_month` | 68.6% missing. Prefer `hdds_good_season` / `hdds_bad_season`. |
| Philippines and Gambia | `food_shortage_flag` | Zero valid responses — exclude from food shortage rate charts. |
| Vulnerability profile nulls | `vulnerability_profile_k4` | 23.8% unclassified. Shown as "No Profile Data" in dashboards — valid missing, not error. |
| Income negative values | `total_income_ppp_per_year`, `income_ppp_per_mae` | Small number of negative values from PPP conversion artefacts. Exclude from income distribution charts. |

---

## Canonical Source of Truth

Cleaning logic: `scripts/etl_pipeline.py`
Analysis-phase derivations: `notebooks/04_statistical_analysis.ipynb`, `notebooks/05_final_load_prep.ipynb`
KPI definitions: see `notebooks/05_final_load_prep.ipynb` Section 2
