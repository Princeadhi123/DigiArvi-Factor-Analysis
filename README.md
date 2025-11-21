# DigiArvi CPFQ Factor Analysis

## Overview

This repository contains Python scripts for running descriptive and factor-analytic workflows on the DigiArvi CPFQ dataset. The analyses focus on:

- Descriptive statistics and visualizations for key questionnaire scores and demographic variables.
- Exploratory factor analysis (EFA) of a subset of CPFQ items.
- Parallel analysis to help decide how many factors to retain.

All scripts assume that the main data file is available as an Excel workbook in the project root.

## Repository structure

- `CPFQ_REVERSED_USETHIS.xls`  
  Main dataset used by all analysis scripts (Excel `.xls` format).

- `metadata.xlsx`  
  Optional metadata file (for example, variable descriptions or coding information if provided).

- `descriptive_analysis.py`  
  End-to-end descriptive analysis pipeline for the DigiArvi dataset:
  - Loads the Excel file.
  - Computes extended numerical summaries (mean, SD, skewness, kurtosis, IQR, missingness, etc.).
  - Summarizes distributions of key categorical variables (e.g., `sex`, `school_lang`, `home_lang`, `strong_lang`, `friend_lang`, `grade`).
  - Computes a correlation matrix for all numeric variables.
  - Produces multiple publication-ready plots.

- `cpfq_factor_analysis.py`  
  End-to-end factor analysis pipeline for a subset of CPFQ items:
  - Loads the Excel file.
  - Selects CPFQ items (including reversed versions where applicable).
  - Examines missing values and pairwise complete observations.
  - Computes KMO and Bartlett’s test.
  - Runs factor analysis (3-factor solution, oblimin rotation) using `factor_analyzer`.
  - Saves numerical output and multiple visualization files.

- `parallel_analysis.py`  
  Parallel analysis workflow to support factor-retention decisions:
  - Imports data-loading and preparation helpers from `cpfq_factor_analysis.py`.
  - Generates many random datasets with the same dimensions as the prepared CPFQ data.
  - Compares eigenvalues from the real data vs. random data.
  - Produces a scree plot and a text summary recommending a number of factors.

- `results_descriptive/`  
  Default output directory for `descriptive_analysis.py` (created automatically if missing).

- `results_cpfq/`  
  Default output directory for factor and parallel analysis scripts (created automatically if missing).

## Requirements

Use a recent Python 3 version and install the following Python packages (via `pip`, `conda`, etc.):

- `pandas`
- `numpy`
- `matplotlib`
- `seaborn`
- `scipy`
- `factor_analyzer` (provides `FactorAnalyzer`, KMO, and Bartlett tests)
- `xlrd` (Excel reader; required because `pd.read_excel(..., engine="xlrd")` is used)

Example (using `pip`):

```bash
pip install pandas numpy matplotlib seaborn scipy xlrd
pip install factor_analyzer
```

If you use a different Excel engine (e.g. `openpyxl`), you can adjust the `load_data` functions accordingly.

## Data expectations

All scripts expect to be run from the project root so that relative paths resolve correctly.

- The main data file is referenced as:
  - `DATA_FILE = 'CPFQ_REVERSED_USETHIS.xls'`
- Output directories are referenced as:
  - In `descriptive_analysis.py`: `RESULTS_DIR = 'results_descriptive'`
  - In `cpfq_factor_analysis.py`: `RESULTS_DIR = 'results_cpfq'`
  - In `parallel_analysis.py`: `RESULTS_DIR` is imported from `cpfq_factor_analysis.py`.

Columns referenced in the scripts include (non-exhaustive examples):

- CPFQ items (e.g. `cpfq_1`, `cpfq_2`, `cpfq_3`, `cpfq_4`, `cpfq_7`, `cpfq_8`, `cpfq_10`, `cpfq_11`, `cpfq_14`, and reversed items like `cpfq_5_rev`, `cpfq_6_rev`, `cpfq_9_rev`, etc.).
- Score variables such as `SCORE_SRF`, `SCORE_F3.2`, and `T10_TOTAL_theta_25`.
- Demographic / categorical variables such as `sex`, `school_lang`, `home_lang`, `strong_lang`, `friend_lang`, and `grade`.

Make sure your dataset contains the variables required by the relevant script(s).

## How to run the analyses

All commands below assume you are in the project root directory (`DigiArvi-Factor-Analysis`).

### 1. Descriptive analysis

Run:

```bash
python descriptive_analysis.py
```

What it does:

- Loads `CPFQ_REVERSED_USETHIS.xls`.
- Computes numerical descriptive statistics for all numeric variables (including skewness, kurtosis, IQR, and missingness details).
- Summarizes the distributions of key categorical variables.
- Computes a correlation matrix for numeric variables.
- Creates multiple figures (score distributions, categorical distributions, correlations, and score comparisons).

Key outputs (written to `results_descriptive/`):

- `descriptive_statistics.txt`  
  Combined numerical statistics, categorical distributions, and correlation matrix, plus a brief guide to interpreting correlations.

- `score_distributions.png`  
  Histograms (with KDE) for `SCORE_SRF` and `SCORE_F3.2`, including mean/median markers.

- `categorical_distributions.png`  
  Count plots (with percentage labels) for the main categorical variables.

- `correlation_heatmap.png`  
  Correlation matrix heatmap for selected numeric variables.

- `scores_by_sex.png`, `scores_by_grade.png`, `scores_by_school_lang.png`  
  Boxplots comparing `SCORE_SRF` and `SCORE_F3.2` across selected categorical variables.

The script prints a summary of saved files when it finishes.

### 2. CPFQ factor analysis

Run:

```bash
python cpfq_factor_analysis.py
```

Main steps:

1. **Load data** using `load_data()` from `CPFQ_REVERSED_USETHIS.xls`.
2. **Prepare CPFQ items** with `prepare_data()`:
   - Prints available CPFQ columns.
   - Selects items `[1, 2, 3, 4, 7, 11, 12, 13, 15]`, using reversed versions where appropriate (e.g., `cpfq_1` vs. `cpfq_1_rev`).
3. **Missing-data and pairwise analysis** with `analyze_missing_and_pairs(...)`:
   - Detailed missing-value table for each CPFQ item.
   - Pairwise complete-case counts and summary statistics.
4. **KMO** with `calculate_kmo(...)`:
   - Overall sampling adequacy.
   - KMO values for each CPFQ variable.
5. **Bartlett’s test** with `perform_bartlett_test(...)`:
   - Tests suitability of the correlation matrix for factor analysis.
6. **Correlation matrix and scree plot**:
   - `plot_correlation_matrix(...)` for CPFQ items.
   - `create_scree_plot(...)` based on eigenvalues.
7. **Factor analysis** with `perform_factor_analysis(...)`:
   - 3-factor solution using `FactorAnalyzer` (principal method, `oblimin` rotation).
   - Factor loadings and variance explained.
   - Interprets and labels factors (e.g., experiential avoidance vs. fusion, committed action, present-moment awareness & values) and lists items with significant loadings.
8. **Visualizations**:
   - Heatmap of factor loadings.
   - Additional factor-importance plot.
9. **Save combined results** with `save_results(...)` and print a summary of output files.

Key outputs (written to `results_cpfq/`):

- `cpfq_factor_analysis_results.txt`  
  Comprehensive text report including:
  - Missing-value analysis.
  - KMO and Bartlett results.
  - Factor loadings table.
  - Variance explained.
  - Narrative interpretations of the three factors.

- `correlation_matrix.png`  
  Heatmap of CPFQ item correlations (lower triangle shown, annotated).

- `scree_plot.png`  
  Scree plot of eigenvalues for CPFQ items.

- `factor_importance.png`  
  Bar plot showing the percentage of variance explained by each retained factor.

- `factor_loadings_heatmap.png`  
  Heatmap of factor loadings for CPFQ items across the three factors.

The script prints a list of all saved files upon completion.

### 3. Parallel analysis for factor retention

Run:

```bash
python parallel_analysis.py
```

Workflow:

1. Uses `load_data()` and `prepare_data()` from `cpfq_factor_analysis.py` to obtain the CPFQ subset.
2. Computes eigenvalues of the real-data correlation matrix.
3. Runs **parallel analysis** by generating many random datasets with the same shape as the CPFQ data:
   - Controlled by `ITERATIONS` (default 1000) and `PERCENTILE` (default 95).
4. Compares actual eigenvalues to the chosen percentile of random eigenvalues.
5. Plots both curves in a parallel-analysis scree plot.
6. Counts how many actual eigenvalues exceed the threshold and reports the suggested number of factors.
7. Saves a text report and the scree plot.

Key outputs (written to `results_cpfq/`):

- `parallel_analysis_results.txt`  
  Text report summarizing method settings and, for each factor, the actual eigenvalue and whether it should be retained. Includes the final recommended number of factors and a warning if none meet the criteria.

- `parallel_analysis_scree_plot.png`  
  Scree plot comparing actual vs. random eigenvalues with a visual marker at the suggested cutoff.

The script prints the suggested number of factors and saved-file locations when it completes.

## Configuration and customization

You can adjust key settings directly in the scripts if needed.

### In `descriptive_analysis.py`

- `RESULTS_DIR` – where text and figures are saved (default: `results_descriptive`).
- `DATA_FILE` – path to the Excel data (default: `CPFQ_REVERSED_USETHIS.xls`).
- Plotting settings:
  - `PLOT_STYLE`, `PLOT_CONTEXT` (global seaborn style/context).
  - `DIST_PALETTE`, `CAT_PALETTE`, `CORR_PALETTE`, `COMP_PALETTE` for color palettes.
- Variable selections:
  - Categorical variables listed in `categorical_cols`.
  - Variables included in the correlation heatmap (`selected_vars` list).

### In `cpfq_factor_analysis.py`

- `RESULTS_DIR` – output directory for factor-analysis results (default: `results_cpfq`).
- `DATA_FILE` – Excel data file path.
- Plot settings and palettes for correlation heatmap, scree plot, factor loadings, etc.
- `selected_items` inside `prepare_data(...)` – which CPFQ items (and/or reversed items) are included in the analysis.
- `perform_factor_analysis(...)` – number of factors (`n_factors=3`), rotation method (`'oblimin'`), and extraction method (`'principal'`). You can change these according to your analysis plan.

### In `parallel_analysis.py`

- `ITERATIONS` – number of random datasets to generate (higher values give more stable estimates but take longer).
- `PERCENTILE` – percentile of random eigenvalues used as a threshold (commonly 95th).
- `FIGURE_DPI`, `FIGURE_FORMAT` – quality and format of the scree plot.

If you require fully reproducible random results for the parallel analysis, you can introduce a fixed random seed (e.g., via `np.random.seed(...)`) before generating random data.

## Troubleshooting

- **`FileNotFoundError: CPFQ_REVERSED_USETHIS.xls`**  
  Make sure the Excel file is in the project root and that you run the scripts from that directory. If the file name or location differs, update `DATA_FILE` in each script.

- **`ModuleNotFoundError` for `pandas`, `factor_analyzer`, etc.**  
  Install the missing package with `pip` or your preferred environment manager.

- **Excel engine issues (e.g. `xlrd` errors)**  
  Ensure `xlrd` is installed. For newer versions of pandas, you may prefer a different engine (e.g. `openpyxl`) and update the `pd.read_excel` call accordingly.

- **Plots not showing up interactively**  
  The scripts save plots directly to files; they do not open interactive windows by default. Check the relevant `results_*` directory for PNG files.