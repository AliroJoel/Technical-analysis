# README — How to use `replication_package.py` in Google Colab with `df.xlsx`

This repository contains the script `replication_package.py` and the Excel file `df.xlsx` with all necessary sheets to run:

- **In-sample** and **Out-of-sample**:  
  `Fundamentals`, `Macroeconomics`, `Sentiment`, `Technicals`  
  *(The `Appendix` sheet is not used in the code)*

- **Trading strategies**:  
  `Trading_Fundamentals`, `Trading_Macroeconomics`, `Trading_Sentiment`, `Trading_Technicals`

> **Minimum column requirement:** The sheets used by the code include the target series (e.g., `y_f`, `y_f7`, `y_f14`, `y_f28`) and predictors prefixed with `x` (such as `x1`, `x2`, …). For trading sheets, an additional column `Treasury` (risk-free rate) is also required.

---

## 1) Setup in Google Colab

1. Open a new notebook in Colab.
2. Upload the files:
   ```python
   from google.colab import files
   files.upload()  # select replication_package.py and df.xlsx
   ```
   (They will be saved as `/content/replication_package.py` and `/content/df.xlsx`).
3. (Optional) install missing packages:
   ```python
   !pip install statsmodels openpyxl joblib
   ```
4. Run the script by sections (recommended) by copy-pasting the relevant blocks **or** run the entire file after adjusting file paths. Below you will find instructions on what to edit in each section to ensure the correct sheet is loaded.

---

## 2) In-Sample Exercises (descriptive statistics and in-sample tables)

**Where to edit in the code:** section labeled `## in-Sample Exercises`.

- **What to load:** one of the sheets  
  `Fundamentals`, `Macroeconomics`, `Sentiment`, `Technicals`.

- **What to change:** add `sheet_name=...` to `pd.read_excel`.

**Example (choose the sheet you want to run):**
```python
# --- IN-SAMPLE ---
excel_file = "/content/df.xlsx"
SHEET = "Technicals"  # or "Fundamentals", "Macroeconomics", "Sentiment"

import pandas as pd
df = pd.read_excel(excel_file, sheet_name=SHEET)

# the rest of the in-sample block remains unchanged
```

**Expected outputs:**
- `In_Sample.xlsx` with sheets `full_results` and `tables`.  
- If Excel writing fails, CSV files are created:  
  `In_Sample_full_results.csv`, `In_Sample_tables.csv`, `In_Sample_skipped.csv`.
- For some descriptive statistics, an additional file such as `descriptivos_Technicals.xlsx` may be generated (if you run the Technicals sheet).

> **Tip:** to run all 4 predictor sets, repeat the block changing `SHEET`.

---

## 3) Out-of-Sample Exercises (OOS)

**Where to edit:** section `##Out-of-sample Exercises`.

- **What to load:** one of the sheets  
  `Fundamentals`, `Macroeconomics`, `Sentiment`, `Technicals`.

- **What to change:** add `sheet_name=...` and adjust parameters if needed.

**Example:**
```python
# --- OUT-OF-SAMPLE ---
excel_file = "/content/df.xlsx"
SHEET = "Macroeconomics"  # or "Fundamentals", "Sentiment", "Technicals"

import pandas as pd
df = pd.read_excel(excel_file, sheet_name=SHEET)

# Key parameters (adjust if the effective sample is too short):
START_RATIO = 0.5   # OOS starting point (use 0.3 if dataset is small)
MIN_OBS     = 10    # minimum number of observations for training
HAC_LAGS    = 1     # lags for HAC correction in Clark–West

# run the rest of the OOS block unchanged
```

**Expected outputs:** metrics printed in console and Excel/CSV files depending on the block. If no Excel file is created, check the logs — the script falls back to CSV.

---

## 4) Trading Strategy — Anatoliev & Gerko (2005)

**Where to edit:** section `## Trading Strategy: Anatoliev and Gerko`.

- **What to load:** one of the **trading** sheets  
  `Trading_Fundamentals`, `Trading_Macroeconomics`, `Trading_Sentiment`, `Trading_Technicals`.

- **What to change:** update `EXCEL_PATH` to `df.xlsx` and add `sheet_name`.

**Example:**
```python
# --- TRADING: ANATOLIEV & GERKO (2005) ---
EXCEL_PATH = "/content/df.xlsx"
SHEET = "Trading_Technicals"  # or "Trading_Fundamentals", "Trading_Macroeconomics", "Trading_Sentiment"

import pandas as pd
df = pd.read_excel(EXCEL_PATH, sheet_name=SHEET)

# Typical parameters for this block:
START_RATIO = 0.5   # initial sample size for rolling OOS
MIN_OBS     = 10

# run the rest of the block unchanged
```

**Expected outputs (depending on code):**
- `In_Sample_SEP.xlsx` with sheets `sep_results` and `strategies_timeseries`.  
- If Excel writing fails, fallback CSV files:  
  `In_Sample_SEP_results.csv`, `In_Sample_SEP_strategies.csv`.

---

## 5) Trading Strategy — Wang et al. (2020)

**Where to edit:** section `## Trading Strategy: Wang et al.(2020)`.

- **What to load:** one of the **trading** sheets  
  `Trading_Fundamentals`, `Trading_Macroeconomics`, `Trading_Sentiment`, `Trading_Technicals`.

- **What to change:** same as above, with Wang-specific parameters.

**Example:**
```python
# --- TRADING: WANG ET AL. (2020) ---
excel_file = "/content/df.xlsx"
SHEET = "Trading_Fundamentals"  # or "Trading_Macroeconomics", "Trading_Sentiment", "Trading_Technicals"

import pandas as pd
df = pd.read_excel(excel_file, sheet_name=SHEET)

# Hyperparameters used in this block:
start_ratio = 0.5   # initial sample size
gamma = 6           # risk-aversion / penalty parameter
tau = 0.001         # transaction cost or threshold, depending on implementation

# run the rest of the block unchanged
```

**Expected outputs:** dataframes/files with results and strategy time series (check logs for file names; if Excel fails, fallback CSVs are generated).

---

## 6) Common Errors and Fixes

- **`[ERROR] Target column 'y_f' not found`**  
  You are not using the correct sheet or `sheet_name` was not passed to `pd.read_excel`.  
  → Verify `SHEET` and ensure the sheet contains `y_f`.

- **`[ERROR] No predictors detected with prefix 'x'`**  
  The sheet has no predictor columns starting with `x`.  
  → Use a valid sheet or check that variable names retain the `x` prefix.

- **Too many `NaN` values or short sample**  
  → Lower `START_RATIO` (e.g., 0.3) or `MIN_OBS` (e.g., 5) to start earlier.

- **`Benchmark MSE = 0` or constant series**  
  → Ensure `y_f` has variation and is not constant within the sample.

- **Excel file not created**  
  → The script automatically falls back to `.csv`. Download the CSV files instead.

---

## 7) Recommended Workflow (quick)

1. **In-sample**: run 4 times changing `SHEET` between  
   `Fundamentals`, `Macroeconomics`, `Sentiment`, `Technicals`.  
2. **Out-of-sample**: repeat the same with OOS section (adjust `START_RATIO` if needed).  
3. **Trading (A&G and Wang)**: use the `Trading_*` sheets and run each strategy block.

---

## 8) Downloading Results

The script already attempts:
```python
from google.colab import files
files.download("In_Sample.xlsx")
# and other files as they are generated
```
If no download dialog appears, check the *Files* panel (left side) in Colab and download manually.
