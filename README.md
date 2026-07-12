# Global Port Traffic Analysis (PySpark / Databricks)

Exploratory analysis of global shipping port traffic data using PySpark on Databricks. Course project — Big Data, SGH Warsaw School of Economics, 1st semester.

## Status

✅ **Runs end-to-end without errors**, verified top to bottom on Databricks.

## Dataset

`Port_Data.csv` — 480 records, one row per port/anchorage, with columns:

| Column | Description |
|---|---|
| `Country` | Country the port belongs to |
| `Port_Name` | Name of the port or anchorage |
| `UN_Code` | UN/LOCODE identifier |
| `Vessels_in_Port` | Vessel count currently in port |
| `Departures_Last24Hours` | Departures in the last 24 hours |
| `Arrivals_Last24Hours` | Arrivals in the last 24 hours |
| `Expected_Arrivals` | Expected arrivals |
| `Type` | Port or Anchorage |
| `Area_Local` / `Area_Global` | Regional groupings |
| `Also_known_as` | Alternate names/aliases |

Data is a point-in-time snapshot, not a time series — there's no date column, so "last 24 hours" reflects when the snapshot was pulled.

## What this project does

- Loads the dataset into a Spark DataFrame and registers it as a temp SQL view
- Explores traffic patterns via Spark SQL: max/avg departures and arrivals by country and port, port counts by country, arrivals vs. expected arrivals by country
- Aggregates and visualizes total vessels in port by country (matplotlib bar chart)
- Attempts a linear regression (PySpark ML) predicting `Expected_Arrivals` from vessel/traffic features, with a planned hyperparameter search via `TrainValidationSplit`

## Results

Two linear regression models were fit using PySpark ML, evaluated on a held-out test split (seeded for reproducibility):

| Model | Features | Target | RMSE | R² |
|---|---|---|---|---|
| Multi-feature | `Vessels_in_Port`, `Departures_Last24Hours`, `Arrivals_Last24Hours` | `Expected_Arrivals` | 96.63 | 0.37 |
| Single-feature | `Departures_Last24Hours` | `Arrivals_Last24Hours` | 23.88 | — |

**Interpretation:** the multi-feature model explains only ~37% of the variance in `Expected_Arrivals` (R² = 0.37) — a modest fit, not a strong one. This is a reasonable and expected result given the small sample (480 rows) and the fact that port-level "expected" traffic likely depends on factors not captured here (scheduled vessel calls, seasonal trade patterns, port capacity), rather than just current traffic snapshots. The single-feature departures→arrivals model has a much lower RMSE, but that's largely because departures and arrivals are naturally correlated at any given port (busy ports have both), not because it's a stronger predictive model — R² wasn't computed for it, so it isn't directly comparable to the multi-feature result above.

## What this project does not do (yet)

- No written interpretation embedded in the notebook itself — the results above exist in this README but not yet as markdown cells inline
- No feature engineering beyond raw counts (e.g. ratios, port-type indicators)
- No cross-validation — single train/test split only

## Setup

This notebook was written for **Databricks** (uses `%sql` magic and `display()`, which are Databricks-only). To run it elsewhere, you'll need to either:

**Option A — Run on Databricks (recommended, matches original environment)**
1. Import the `.ipynb` into a Databricks workspace
2. Upload `Port_Data.csv` to DBFS (e.g. `/FileStore/tables/Port_Data.csv`)
3. Attach the notebook to a cluster with Spark 3.x and run top to bottom

**Option B — Run locally with PySpark**
1. Replace `%sql` cells with `spark.sql("...")` calls
2. Replace `display(df)` with `df.show()`
3. Update the file path to a local path relative to the repo, e.g.:
   ```python
   file_location = "./data/Port_Data.csv"
   ```
4. Install dependencies and run:
   ```bash
   pip install -r requirements.txt
   jupyter notebook "Final_Project.ipynb"
   ```

### requirements.txt
```
pyspark>=3.4
matplotlib>=3.7
numpy>=1.24
scikit-learn>=1.3
pandas>=2.0
```

## Repo structure
```
.
├── README.md
├── requirements.txt
├── data/
│   └── Port_Data.csv
└── Final_Project.ipynb
```

## Author

Ahmed Abu Deiab — www.linkedin.com/in/ahmed-abu-deiab · Cost of Living Analyst, Mercer · MSc Advanced Analytics – Big Data, Warsaw School of Economics SGH
