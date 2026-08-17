# BRFSS–NYTS Interoperable Pipeline

A reproducible, interoperable ETL and feature-engineering framework for CDC-family public health surveillance data, built on a Medallion Lakehouse pattern in Databricks and explicitly aligned with [CDC's Data Modernization Initiative](https://www.cdc.gov/surveillance/data-modernization/index.html).

This project harmonizes two structurally distinct federal surveillance systems — an adult health survey and a youth tobacco survey, from two different agencies, in two different file formats — into a single common schema, and demonstrates that one reusable query function can compute correct, validated statistics against either source.

## Why this exists

Most public health ML/data science projects process one dataset well. This project asks a different question: **can a single framework genuinely harmonize CDC surveillance data across systems**, not just process one survey competently? The two sources were deliberately chosen for their structural dissimilarity — different administering agency, different file format, different missing-data convention, different population — to make that claim testable rather than assumed.

## Data sources

| Source | Population | Format | n | Agency |
|---|---|---|---|---|
| [2024 BRFSS](https://www.cdc.gov/brfss/annual_data/annual_2024.html) | Adults (18+) | Fixed-width ASCII | 457,670 | CDC |
| [2025 NYTS](https://www.fda.gov/media/191376) | Youth (middle/high school) | SAS7BDAT | 23,630 | FDA |

## Pipeline structure

| Notebook | Produces |
|---|---|
| `01_bronze_ingestion_brfss.ipynb` | Raw BRFSS parsing (301-variable fixed-width layout) |
| `02_bronze_ingestion_nyts.ipynb` | Raw NYTS parsing (SAS7BDAT via in-memory byte buffer) |
| `03_gold_harmonization_brfss_nyts.ipynb` | Both sources mapped into a common schema; reusable cross-system prevalence function |
| `04_gold_schema_crosswalk_metadata.ipynb` | Machine-readable documentation of every variable mapping |
| `05_pipeline_lineage_metadata.ipynb` | Full source-to-Gold traceability record |

Each stage follows Bronze (raw) → Silver (cleaned, typed, validated) → Gold (harmonized, analysis-ready) conventions.

## Validation

Every statistic produced by this pipeline was checked against an independent, external, published reference before being trusted:

| Check | Result | Reference |
|---|---|---|
| BRFSS row count | 457,670 | Exact match to CDC's published total |
| BRFSS Tennessee exclusion | Correctly absent | Matches CDC's documented data-collection shortfall |
| BRFSS national inactivity rate (unweighted, matched sample) | 22.51% | Matches source manuscript's 22.5% (Δ=0.01pt) |
| BRFSS INCOME3 missingness | 87,423 nulls | Exact match to source manuscript's cited figure |
| NYTS weighted e-cigarette use rate | 5.23% | Matches FDA's published 5.2% (Δ=0.03pt) |

## Two documented data-quality findings

**1. A CDC layout discrepancy.** The BRFSS file's actual record length (2,061 characters) was one variable short of CDC's documented 2,111-position layout — `_AIDTST4` (HIV testing history) is absent from the file. CDC's own BRFSS page discloses the dataset "has been modified to comply with [2025] executive orders," which is the likely explanation. Documented here rather than silently worked around.

**2. A caught-and-corrected pipeline bug.** An early NYTS cleaning pass produced a weighted estimate of 5.18% instead of the correct 5.23%, traced to a type-casting artifact (`astype(str)` silently converting real null values into the literal string `"nan"`, bypassing missing-value recoding for 250 respondents). Caught only via cross-validation against an independent Bronze-layer check — a concrete illustration of why validated pipelines matter, not just an incidental bug.

## Reproducibility artifacts

Two Delta tables accompany the data itself:

- `gold_schema_crosswalk_metadata` — every source-to-harmonized variable mapping, transformation logic, and missing-value convention, with each entry flagged as independently verified against its source codebook or not.
- `gold_pipeline_lineage` — for every table in the pipeline: source, retrieval date, row count, upstream dependency, and known issues.

## Related work

This project extends [BRFSS Physical Inactivity Analysis (2024)](https://prakashsilwal.github.io/brfss-inactivity-2024), submitted to *Preventing Chronic Disease* (CDC).

## Author

Prakash Silwal — MS Data Science, Carolina University. Senior Data Engineer (AWS, Databricks, Snowflake, Spark).
