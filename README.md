# Flight Arrival Delay Prediction

**CS 6220 – Data Mining Techniques, Northeastern University**

## Overview

We predict whether a US domestic flight will arrive 15 or more minutes late (`ArrDel15`) using only information available before departure. The data comes from the Bureau of Transportation Statistics (BTS) *Reporting Carrier On-Time Performance (1987–present)* dataset. We train on 2024, validate on January–June 2025, and test on July–December 2025.

**Team:** [TODO: names]

## Repository Structure

```
.
├── data/
│   ├── raw/              # Monthly BTS CSVs (YYYY_MM.csv), not committed
│   └── processed/        # Parquet files from prepare_data.py, not committed
├── notebooks/            # Exploration and modeling notebooks
├── src/                  # Data preparation and modeling code
├── reports/              # Write-ups, figures, and presentations
├── sample_2024_01.csv    # 5,000-row sample of January 2024 (committed)
└── requirements.txt
```

## Getting the Data

### Option A: Shared drive (recommended)

The full raw data (zipped monthly files named `YYYY_MM.zip`) is on our shared drive: [TODO: link]

Unzip each file into `data/raw/` so you end up with `data/raw/YYYY_MM.csv`.

### Option B: Download from BTS

1. Go to <https://www.transtats.bts.gov/Tables.asp?QO_VQ=EFD> and click **Download** next to *Reporting Carrier On-Time Performance (1987–present)*.
2. Set the filters:
   - **Filter Geography:** All
   - **Filter Year:** the year you want
   - **Filter Period:** one month (each download covers one month)
3. Leave **Prezipped File** unchecked.
4. Uncheck the 6 airport/city-market ID fields that are ticked by default.
5. Tick exactly these fields:

   | Category | Fields |
   |---|---|
   | Time period | `Year`, `Month`, `DayofMonth`, `DayOfWeek`, `FlightDate` |
   | Airline | `Reporting_Airline`, `Tail_Number`, `Flight_Number_Reporting_Airline` |
   | Origin | `Origin`, `OriginCityName`, `OriginState` |
   | Destination | `Dest`, `DestCityName`, `DestState` |
   | Departure | `CRSDepTime`, `DepTime`, `DepDelay`, `DepTimeBlk` |
   | Arrival | `CRSArrTime`, `ArrTime`, `ArrDelay`, `ArrDel15`, `ArrTimeBlk` |
   | Cancellations / diversions | `Cancelled`, `CancellationCode`, `Diverted` |
   | Flight summary | `CRSElapsedTime`, `Distance` |
   | Delay causes | `CarrierDelay`, `WeatherDelay`, `NASDelay`, `SecurityDelay`, `LateAircraftDelay` |

6. Save each month as `data/raw/YYYY_MM.csv` (e.g., `data/raw/2024_01.csv`).

> **Note:** Not all of these fields are used as features. Fields such as `DepTime`, `DepDelay`, `ArrTime`, and the delay-cause columns are only known after departure. We keep them for analysis and labeling, but they must not be used as model inputs.

## Setup

We use a virtual environment in `.venv/` at the repo root (already in `.gitignore`). It was created with Python 3.14 (Homebrew `python3.14`), but Python 3.10+ should work. The macOS built-in `/usr/bin/python3` (3.9) is too old.

```bash
python3 -m venv .venv          # or: python3.14 -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements.txt
```

`requirements.txt` installs pandas, numpy, pyarrow, scikit-learn, lightgbm, xgboost, matplotlib, seaborn, jupyter, and ipykernel.

Optionally, register the environment as a Jupyter kernel:

```bash
python -m ipykernel install --user --name cs6220 --display-name "CS6220 (.venv)"
```

> **Note (macOS):** LightGBM and XGBoost need OpenMP. If importing either fails with a `libomp` error, run `brew install libomp`.

Then prepare the data:

```bash
python src/prepare_data.py   # converts data/raw/*.csv to Parquet in data/processed/
```

To explore the data without downloading anything, use `sample_2024_01.csv`.

## Note on Data

The raw data is not committed because the monthly files exceed GitHub's file size limits. Only the small `sample_2024_01.csv` is tracked in the repo. `.gitignore` excludes `data/`, `*.csv`, `*.zip`, and `*.parquet`, with an exception for `sample_2024_01.csv`.
