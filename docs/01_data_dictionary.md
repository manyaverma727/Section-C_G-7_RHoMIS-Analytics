# Data Dictionary

**Dataset:** RHoMIS (Rural Household Multi-Indicator Survey)  
**Source:** Harvard Dataverse — [DOI: 10.7910/DVN/WS38SA](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi%3A10.7910%2FDVN%2FWS38SA)  
---

## Files

| File | Description | Location |
|---|---|---|
| `rhomis_full_survey_data.csv` | Raw survey responses — 54,873 rows × 1,599 cols | `data/raw/` |
| Cleaned survey | Output of notebook 03_cleaning | `data/processed/` |

---

## Key Column Groups (working subset — ~50 columns)

### Household Demographics
TBA — populated during notebook 01_extraction

### Income & Productivity
TBA

### Food Security (HFIAS)
TBA

### Gender Control Variables
TBA

---

## Derived Columns (engineered in 02_cleaning)

| Column | Formula | Notes |
|---|---|---|
| `income_per_hectare` | `total_income_usd / land_area_ha` | TBA |
| `food_security_category` | Binned from HFIAS score | TBA |
| `farm_size_bucket` | small / medium / large | TBA |

---

## Notes

- Income normalised to USD using exchange rates in the Data Processing Guide
- Country names standardised to ISO 3166-1 alpha-2 codes
- Columns with >70% missing values dropped during cleaning
