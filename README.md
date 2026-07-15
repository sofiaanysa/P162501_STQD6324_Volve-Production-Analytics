# Well Production Performance Analytics for Oil & Gas Operations

### A Data Management & Analytics Case Study Using the Volve Field Open Dataset

| | |
|---|---|
| **Name** | SOFIA ANNISA BINTI JAIRI |
| **Student ID** | P162501 |
| **Course** | STQD6324 Data Management |
| **Semester** | 2, 2025/2026 |
| **Tools** | Apache Hive (data management / ETL layer) + Python via Google Colab (analytics & visualization layer) |

---

## Project Overview

This project builds an end-to-end **data management and analytics pipeline** for upstream oil & gas production surveillance, using the publicly released **Volve field** dataset from Equinor. The pipeline deliberately separates two layers, mirroring how a real production-data stack is structured:

- **Data management / ETL layer — Apache Hive.** The raw daily production file is loaded into a Hive external table, then HiveQL is used to enforce a schema, handle structural missingness, cap/clip anomalous sensor readings, and pre-aggregate a monthly summary per well.
- **Analytics layer — Python (Google Colab).** The Hive-cleaned tables are bridged into pandas (via a thin PySpark read/export step), where analytical features are engineered (water cut, gas–oil ratio, downtime hours, month index) and used to drive exploratory analysis, visualizations, a decline-curve forecast, and written interpretation.

The analysis covers the full producing history of the Volve field (2007–2016) across 7 well bores, and concludes with a set of operational recommendations grounded in the findings.

---

## Repository Structure

```
P162501_STQD6324_Volve_Production_Analytics/
├── P162501_STQD6324_Volve_Production_Analytics.ipynb   # Main Jupyter Notebook
├── Original_Data.csv                                    # Volve daily production data (upload separately)
└── README.md                                            # This file
```

---

## Dataset

| Property | Detail |
|---|---|
| **Name** | Volve field daily production data |
| **Source** | Equinor Open Data — https://www.equinor.com/energy/volve-data-sharing |
| **Field** | Volve, Norwegian North Sea (produced 2008–2016, decommissioned) |
| **Coverage** | 7 well bores, September 2007 – December 2016 |
| **File** | `Original_Data.csv` — daily production sheet derived from `Volve production data.xlsx` |

Key columns include `DATEPRD` (date), `NPD_WELL_BORE_NAME` (well identifier), `ON_STREAM_HRS`, downhole/wellhead pressure and temperature, choke size, and daily oil/gas/water/water-injection volumes (`BORE_OIL_VOL`, `BORE_GAS_VOL`, `BORE_WAT_VOL`, `BORE_WI_VOL`).

Two of the seven wells (`15/9-F-4`, `15/9-F-5`) are water injectors used ahead of first oil rather than producers, and several producing wells came online later (2013–2014) — both facts are accounted for in the cleaning and interpretation.

---

## Methodology

### 1. Introduction & Industry Context
Frames the project within upstream oil & gas production surveillance, predictive maintenance, and reservoir analytics.

### 2. Dataset Overview & Source
Describes the Volve field, Equinor's open data release, and the structure of the daily production file.

### 3. Environment & Tools Setup (Google Colab required)
Installs and configures a single-machine **Hadoop 3.3.6 + Hive 3.1.3** stack running in local mode (no real HDFS/YARN cluster needed), on **Java 8** (Hive 3.1.3 is incompatible with Java 11). Also installs the Python analytics stack: pandas, numpy, matplotlib, seaborn, plotly.

### 4. Data Collection & Understanding
First look at `Original_Data.csv`: shape, schema, missing-value audit, and well operating windows. Establishes that the large blocks of missing pressure/volume readings are **structurally** missing (e.g. injector-only fields), not random.

### 5. Data Management Layer — Apache Hive
The core ETL component, run as HiveQL scripts:
- **`volve_raw`** — external table over the raw CSV
- **`volve_clean`** — schema-enforced, cleaned table (nulls → 0 for legitimate no-flow days, `on_stream_hrs` capped at 24, negative volumes clipped to 0)
- **`volve_monthly_summary`** — monthly roll-up per well/flow type

**PySpark is used purely as a bridge**: it connects to the same embedded Derby metastore Hive just wrote to, reads `volve_clean` and `volve_monthly_summary`, and exports them to flat CSV so Section 6 can load them in pandas. No additional cleaning or aggregation logic is introduced at this step.

### 6. Data Cleaning & Preprocessing (Python — Feature Engineering)
Reads the Hive-cleaned export (falling back to the raw CSV if the Hive export isn't available) and derives analytical features: water cut, gas–oil ratio, downtime hours, and a month index.

### 7. Exploratory Data Analysis & Visualizations
| # | Analysis |
|---|---|
| 7.1 | Field-level monthly oil production (decline trend) |
| 7.2 | Total oil produced per well |
| 7.3 | Water cut trend by well (reservoir depletion signal) |
| 7.4 | On-stream hours vs. daily oil volume |
| 7.5 | Correlation structure of sensor variables (heatmap) |
| 7.6 | Production vs. injection volume balance |
| 7.7 | Well downtime ranking (raw cumulative hours) |
| 7.7b | Relative downtime — % of each well's own days offline (longevity-adjusted) |
| 7.8 | Interactive Plotly view: monthly production by well |
| 7.9 | Arps hyperbolic decline-curve forecast (bonus) for the field's two largest producers |

Each chart is paired with a short written insight.

### 8. Insights & Explanations
Synthesizes the individual chart findings: the field is in structural (waterflood) decline, output is concentrated in two wells, and downtime is a major controllable lever.

### 9. Recommendations
Operational recommendations, including prioritising an uptime-improvement programme on the top producers and setting up automated downtime/anomaly alerting.

### 10. Conclusion
Summarizes the pipeline and headline findings, including a field-wide voidage-replacement ratio of ~1.20.

### 11. References
Sources for the Volve dataset and the Hive/Spark/pandas tooling used.

---

## Hive Tables

Database: `volve`

| Table | Description |
|---|---|
| `volve_raw` | External table over the untouched raw CSV |
| `volve_clean` | Schema-enforced, cleaned daily records (missingness handled, anomalies capped/clipped) |
| `volve_monthly_summary` | Monthly per-well roll-up of oil/gas/water/injection volumes and average downtime |

---

## How to Reproduce

### Google Colab (Required)

This notebook must be run in **Google Colab**, not locally — it needs a fresh Ubuntu VM with full internet access to download Hadoop and Hive.

1. Open `P162501_STQD6324_Volve_Production_Analytics.ipynb` in [Google Colab](https://colab.research.google.com/)
2. Upload `Original_Data.csv` using the 📁 sidebar (or the upload cell in Section 4), so it lands in `/content/`
3. Run all cells in order (`Runtime → Run all`)

> After any runtime restart, always re-run from Cell 1 — the Hadoop/Hive/Java environment variables do not survive a reset.

### Reproducibility Notes
- Section 3 installs Java 8 (required for Hive 3.1.3 compatibility) and a local-mode Hadoop 3.3.6 + Hive 3.1.3 stack with an embedded Derby metastore — no manual cluster setup is needed.
- Section 5 includes defensive checks that re-detect `HADOOP_HOME` if a cell is re-run out of order or after a partial restart.
- Section 6 falls back to reading the raw CSV directly if the Hive export files aren't found, so the notebook remains reproducible even before the Hive pipeline has been run.

---

## Dependencies

| Package | Purpose |
|---|---|
| Apache Hadoop 3.3.6 | Local-mode filesystem backing Hive |
| Apache Hive 3.1.3 | Schema enforcement, cleaning, monthly aggregation (ETL layer) |
| `pyspark==3.4.1` | Bridge: reads Hive tables from the shared metastore, exports to CSV |
| `pandas`, `numpy` | Feature engineering and analysis in the Python layer |
| `matplotlib`, `seaborn` | Static visualizations |
| `plotly` | Interactive monthly-production chart |
| `scipy` (`curve_fit`) | Arps hyperbolic decline-curve fitting |

---

## Environment

| Component | Version | Notes |
|---|---|---|
| Python | 3.11 | Google Colab default |
| Java | OpenJDK **8** | ⚠️ Required — Hive 3.1.3 has known `ClassCastException` issues under Java 11 |
| Hadoop | 3.3.6 | Configured in local (standalone) mode, no HDFS/YARN cluster |
| Hive | 3.1.3 | Embedded Derby metastore, local warehouse directory |
| PySpark | 3.4.1 | Used only as a read/export bridge from Hive to CSV |

> **⚠️ Java Version Warning**
> Hive 3.1.3 requires Java 8. Running it under Java 11 causes `ClassCastException` errors during metastore initialisation. The notebook installs Java 8 explicitly and points `JAVA_HOME` at it before Hadoop/Hive start.

---

## Notebook Structure

| Section | Description |
|---|---|
| 1 | Introduction & industry context |
| 2 | Dataset overview & source |
| 3 | Environment & tools setup (Java 8, Hadoop, Hive, Python libraries) |
| 4 | Data collection & understanding |
| 5 | Data management layer: Hive ETL (raw → clean → monthly summary) + Spark bridge to CSV |
| 6 | Data cleaning & feature engineering in Python |
| 7 | Exploratory data analysis & visualizations (7.1–7.9) |
| 8 | Insights & explanations |
| 9 | Recommendations |
| 10 | Conclusion |
| 11 | References |
