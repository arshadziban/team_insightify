# Analysis Mind Map

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Insightify** analyzes the impact of screen time on mental health and well-being in adolescents, using survey data (452 respondents, ages 11-19) pulled from a Google Sheet. The pipeline runs the data through cleaning, cross-tabulation, chi-square significance screening, and multinomial logistic regression to answer:

- How does daily screen time correlate with depression severity (PHQ-9 scores)?
- What demographic, physical, and behavioral factors predict screen usage patterns?
- Which variables show statistically significant associations with mental health outcomes?

## Setup

- Python 3.11+, virtual environment already configured in `env/`
- `env\Scripts\activate` (Windows) or `source env/bin/activate` (macOS/Linux)
- `pip install -r requirements.txt`
- `.env` needs `GOOGLE_SHEET_URL` pointing at a publicly shareable Google Sheet (git-ignored)

## Pipeline at a Glance

Notebooks run in numeric order; each reads the previous stage's CSV output. `outlier_detection.ipynb` is independent (loads its own data from Google Sheets) and can run any time.

| # | Notebook | Reads | Writes | Purpose |
|---|----------|-------|--------|---------|
| 1 | `1_eda.ipynb` | Google Sheet | `dataset_v2.csv` (452×38) | Clean, engineer features, score PHQ-9 |
| 2 | `2_cross_tab_analysis.ipynb` | `dataset_v2.csv` | `dataset_v3.csv` (452×35) + `report_v2/`, `report_v2.2/` | Merge sparse categories, cross-tab PDF reports |
| 3 | `3_sig_analysis.ipynb` | `dataset_v3.csv` | `dataset_sig_st.csv` (452×20), `dataset_sig_phq9.csv` (452×14) | Chi-square screening, keep p<0.001 predictors |
| 4 | `4_mlr_st.ipynb` | `dataset_sig_st.csv` | `report_v3/*_st.pdf` | MLR: predict daily screen time |
| 5 | `5_mlr_phq9.ipynb` | `dataset_sig_phq9.csv` | `report_v3/*_phq9.pdf` | MLR: predict PHQ-9 depression severity |
| 6 | `6_analysis_for_mind_well.ipynb` | `dataset_v3.csv` | `report_v4/*.pdf` (16 files) | Expanded cross-tabs, 8 outcomes × physical/mental predictors |
| – | `outlier_detection.ipynb` | Google Sheet (direct) | `report_v4/outlier_*.pdf` | IQR + Z-score outlier diagnostics |

**Regenerate everything:** run 1 → 2 → 3, then 4/5/6 selectively.
**New survey data:** update `GOOGLE_SHEET_URL`, re-run from notebook 1.

## Notebook Details

### 1. EDA & Cleaning (`1_eda.ipynb`)
- Loads from Google Sheets via `gsheet-loader`, fixes encoding issues
- Scores PHQ-9 (0-27, 9 items, 4-point scale) into 5 severity bands
- Engineers: age class (11-13 / 14-16 / 17-19), BMI class, income class (Low ≤150K / Middle 150-300K / High >300K), bedtime/wake-time buckets

### 2. Cross-Tab Analysis (`2_cross_tab_analysis.ipynb`)
- Drops low-value columns: `A4_marital`, `E4_addictive_drug`, `F2_screen_impact`, `F3_learning_interest`
- Merges sparse categories to strengthen chi-square power: Religion (Hindu+Christian), Family type (Extended joint → Joint), Device (Laptop+Desktop → Computer/Laptop), BMI (Overweight+Obese), plus regroupings of `B5_time_with_parents_hours`, `B6_relationship_with_family`, `C3_physical_problems`, `D2_weekend_screen`, `D5_parents_screen_time`
- Renames `PHQ_class` "Moderately depressive" → "Moderately Severe depressive"
- Creates `D4_content_grouped` (Educational / Entertainment & Media / Social Media & Mixed / Other)
- Contains dead, commented-out legacy `analysis()` code (not executed)
- Live local `analysis()` function generates two PDF report sets directly from `dataset_v3.csv`:
  - `report_v2.2/` — target `PHQ_class`, 13 sections
  - `report_v2/` — target `D1_daily_screen`, 17 sections (superset, adds residential/economic status)

### 3. Significance Screening (`3_sig_analysis.ipynb`)
- Chi-square test for every variable against `D1_daily_screen` and `PHQ_class`
- Keeps only `***` (p < 0.001) predictors — stricter than the general α = 0.05 used elsewhere
- Outputs the two `dataset_sig_*.csv` filtered datasets plus summary tables of test statistics/p-values

### 4. MLR — Screen Time (`4_mlr_st.ipynb`)
- Predicts `D1_daily_screen` (4 classes: <1hr, 1-3hrs, 3-5hrs, >5hrs)
- Accuracy ~46%; McFadden's pseudo-R² = 0.40 ("very good fit")
- Odds ratios with 95% CIs, goodness-of-fit diagnostics, confusion matrix, feature-importance heatmap
- PDFs to `report_v3/`

### 5. MLR — Depression (`5_mlr_phq9.ipynb`)
- Predicts `PHQ_class` (5 classes: Minimal, Mild, Moderate, Moderately Severe, Severe)
- Same structure as notebook 4, but weaker: accuracy ~37%, McFadden's pseudo-R² = 0.118 ("acceptable fit")
- Zero precision/recall on Moderately Severe and Severe classes — too few respondents in those bins to be predicted in the test split
- Uses Firth's bias-reduced (penalized) logistic regression instead of plain MLE — the unpenalized fit produced quasi-complete separation (astronomical odds ratios/CIs) because the rarest class (n=15) is spread thin across 43 indicator predictor columns. Firth's Jeffreys-prior correction shrinks those coefficients to realistic values without merging categories.
- PDFs to `report_v3/`

### 6. Expanded Cross-Tabs (`6_analysis_for_mind_well.ipynb`)
- Cross-tabulates 8 outcome variables (screen time, weekend screen time, primary device, sleep duration, bedtime, anxious-without-device, feel low/unmotivated, academic satisfaction) against physical- and mental-health predictor sets
- Own local `analysis()` PDF generator (same pattern as notebook 2's, different variable pairs)
- 16 PDFs to `report_v4/`

### Outlier Detection (`outlier_detection.ipynb`)
- Independent of the main pipeline — loads and cleans its own copy of the raw data
- IQR (1.5×IQR fences) and Z-score (|z| > 3) methods for numeric columns; over/under-represented category detection for categorical columns
- Outputs `report_v4/outlier_numeric.pdf`, `report_v4/outlier_categorical.pdf`

## Data Files

| File | Source | Rows × Cols | Purpose |
|------|--------|-------------|---------|
| `dataset_v2.csv` | Notebook 1 | 452 × 38 | Raw cleaned data (intermediate) |
| `dataset_v3.csv` | Notebook 2 | 452 × 35 | Main cleaned dataset (final, used for analyses) |
| `dataset_sig_st.csv` | Notebook 3 | 452 × 20 | Significant predictors of screen time |
| `dataset_sig_phq9.csv` | Notebook 3 | 452 × 14 | Significant predictors of depression |

## Report Outputs

| Directory | Generated by | Contents |
|-----------|--------------|----------|
| `report_v1/` | (legacy) | Initial cross-tabs, 18 sections, earliest variable groupings, `all.zip` bundle |
| `report_v2/` | Notebook 2 | Cross-tabs vs. `D1_daily_screen`, 17 sections + `pvalue_significance_chart_v3.png` |
| `report_v2.2/` | Notebook 2 | Cross-tabs vs. `PHQ_class`, 13 sections + `pvalue_significance_chart_depression_linear.*` |
| `report_v3/` | Notebooks 4, 5 | Odds-ratio tables, MLR publication tables (`*_st.pdf`, `*_phq9.pdf`), feature-importance PNG |
| `report_v4/` | Notebook 6, `outlier_detection.ipynb` | 16 expanded cross-tab PDFs + outlier diagnostics |

Report contents generally include: cross-tab tables with chi-square results, odds ratios with CIs, feature-importance visualizations, outlier diagnostics.

## Key Conventions

- **PHQ-9 bands:** Minimal 0-4, Mild 5-9, Moderate 10-14, Moderately Severe 15-19, Severe 20-27
- **Significance tiers:** `*` p<0.05, `**` p<0.01, `***` p<0.001 (notebook 3's filtered datasets keep `***` only)
- **Odds ratios:** >1 = increased likelihood of outcome, <1 = decreased; MLR uses reference-class encoding
- **Sample size:** 452 respondents, no weighting applied
- **Missing values:** none in the cleaned datasets

**Generate PDF cross-tab reports:** call `analysis()` in notebooks 2/6 with `save_image=True`, specifying `target` and `variables`. In notebook 2, `target='PHQ_class'` → `report_v2.2/`, `target_st='D1_daily_screen'` → `report_v2/`.

## Key Dependencies

| Package | Purpose |
|---------|---------|
| pandas | Data manipulation |
| numpy | Numeric ops (notebooks 4, 5, 6, outlier_detection) |
| scikit-learn | Logistic regression, LabelEncoder |
| scipy.stats | Chi-square tests, Z-score outliers |
| statsmodels | MNLogit with p-values/CIs |
| gsheet-loader | Google Sheets import |
| python-dotenv | `.env` handling |
| matplotlib / seaborn | PDF reports, boxplots, heatmaps |

## Troubleshooting

**Missing `.env`:** create with `GOOGLE_SHEET_URL=https://docs.google.com/spreadsheets/d/...`

**Google Sheets import fails:** confirm the sheet is published/publicly shareable; check URL format against the README example.

**Chi-square warnings (expected frequency < 5):** acceptable with large df; rare categories are already merged in notebook 2.

**MLR accuracy looks low (~46% screen time, ~37% PHQ-9):** expected for 4-5 class multinomial classification. The PHQ-9 model is weaker and has zero precision/recall on its rarest classes (Moderately Severe, Severe) because the test split has too few examples to predict them at all. Judge these models by odds ratios and per-class precision/recall, not overall accuracy.

**Notebook shows stale/inconsistent outputs after an edit:** re-run it end-to-end (`jupyter nbconvert --to notebook --execute --inplace <file>.ipynb`) rather than trusting partially-executed cell outputs — a cell edited without re-running everything downstream leaves old results in place under a new execution count.
