# Data Extraction and Cleaning Methodology
**Project:** RHoMIS Analytics Capstone (Section C, Group 7)

---

## 1. Executive Summary
This document outlines the systematic extraction, transformation, and load (ETL) pipeline constructed for the Rural Household Multiple Indicator Survey (RHoMIS) dataset. The pipeline is designed to transform an exceptionally sparse, high-dimensional raw dataset into a robust analytical datamart optimized for Tableau visualizations and predictive modeling.

---

## 2. Phase 1: Data Extraction & Triage
The original dataset was provided as a compressed, mixed-type CSV (`raw.csv.gz`). Initial programmatic exploratory sweeps revealed a high degree of noise and sparsity.

### Raw Data Profile
- **Starting Dimensions:** 54,873 Rows × 1,599 Columns
- **File Footprint:** ~670 MB
- **Sparsity Crisis:** Over 1,417 columns (88%) registered null densities exceeding 50%.
- **Observation Strategy:** Rather than imputing vast numbers of inherently missing localized variables (e.g., specific micro-livestock parameters per village), the strategy was pivoted to **targeted extraction**. We isolated the core domain indicators vital to fulfilling the overarching business goals: Food Security and Household Income metrics.

### Targeted Column Selection
A subset of crucial columns was retained, categorized into the following domains:
- **Demographics:** `id_unique`, `country`, `region`, `year`, `count_people`, `respondentsex`, `age_malehead`, `age_femalehead`
- **Land & Tenure:** `landcultivated`, `landowned`, `unitland`, `land_tenure`
- **Agricultural Production:** `crop_count`, `crop_name_1`, `crop_yield_1`, `crop_harvest_1`, `crop_consumed_prop_1`
- **Income Sources:** `crop_income_per_year_1`, `livestock_sale_income_1`, `offfarm_incomes_any`
- **Vulnerability Indices:** `hfias_1` through `hfias_9`, `local_currency`

---

## 3. Phase 2: Fundamental Data Cleaning Pipeline
With the dataset constrained to high-value features, a scripted Python ETL pipeline (`scripts/etl_pipeline.py`) was initiated to enforce data integrity.

### 3.1. Standardizing Demographics & Managing Categorical Chaos
Typographical errors and variant inputs were heavily present in text fields.
- **Sex/Gender Harmonization:** The `respondentsex` field contained varying casing and shapes ('m', 'male', 'F', 'Both', etc.). These strings were stripped, lowered, and strictly mapped to uniform binary strings: `['Male', 'Female', 'Both']` alongside null captures.
- **Household Population Safety:** `count_people` contained instances of zero (`0`). Given a household cannot exist with zero population, these were mathematically stabilized to a minimum of `1` using clipping functions. This prevented fatal infinite division errors further down the pipeline.

### 3.2. Outlier Rectification: The Age Phenomenon
- Initial descriptive statistics showcased massive anomalies where max values for `age_malehead` exceeded 19,000. 
- **The Finding:** Respondents incorrectly inputted their *Birth Years* (e.g., 1982) rather than their chronologic age.
- **The Fix:** An algorithmic rule was applied calculating `(2024 - value)` where the value fell between 1900 and 2025. Following standard demographic practices, any remaining extreme mathematical impossibilities (>120 years or <0) were discarded.

### 3.3. Standardizing Spatial Metrics (Area Units)
The survey operated globally, yielding diverse measurements such as Hectares, Acres, Feddan, and Tim.
- **Unification Strategy:** The column `land_cultivated_ha` was instantiated. A conversion algorithm targeted string footprints in `unitland`. Where the string indicated "acre", a multiplier of `0.404686` was applied to convert to Hectares natively.

### 3.4. Spatial Outlier Pruning
An instance was detected presenting a `landcultivated` size exceeding 1,500,000 hectares. To prevent severe skewing in Tableau geospatial and distribution charts, extreme spatial outliers were hard-capped at a 99th-percentile logical constraint (~1,500 Hectares).

---

## 4. Phase 3: Feature Engineering
Transforming raw values into analytical KPIs enabled direct insight generation for Tableau.

### 4.1. Income Mapping
- Independent revenue streams (`crop_income_per_year_1` & `livestock_sale_income_1`) were type-casted to strict numerals.
- Coerced nulls and string-literals were neutralized to `0`.
- **`total_annual_income`** was derived as their direct aggregate constraint.
- **`income_per_capita`** was synthesized mathematically (`total_annual_income` / `count_people`), representing a true household-level economic resilience metric.

### 4.2. Calculating the Food Security Status Matrix
RHoMIS relies on nine distinct Household Food Insecurity Access Scale (HFIAS) indicators (`hfias_1` thru `9`).
- **Severity Weighting:** Raw qualitative responses were mapped onto a linear categorical severity scale:
  - `never`: 0 (No severity)
  - `monthly` / `fewpermonth`: 1 (Low severity)
  - `weekly` / `fewperweek`: 2 (Medium severity)
  - `daily`: 3 (High severity)
- **Cumulative Index:** A unified `hfias_total_score` was constructed (Max severity: 27). 
- **Status Classification:** The score was segmented into human-readable buckets, deriving the pinnacle `food_security_status` categorical feature:
  - `Food Secure` (≤5)
  - `Mildly Insecure` (6-10)
  - `Moderately Insecure` (11-17)
  - `Severely Insecure` (>18)

---

## Conclusion & Output Asset
The final processing script was executed asynchronously over the raw dataset. The output was written explicitly to `data/processed/cleaned_rhomis_data.csv`. 

**Final Export Statistics:**
- **Final Rows:** 54,873 (Zero attrition approach)
- **Final Columns:** 25 (Engineered KPI-heavy matrix)
- **Data Status:** The dataset is statistically constrained, outlier-managed, structurally identical universally, and is cleared for localized Exploratory Data Analysis (EDA) and direct integration into Tableau.
