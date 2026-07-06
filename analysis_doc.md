# Analysis Mind Map

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Insightify** is a data-driven research project analyzing the impact of screen time on mental health and well-being in adolescents. The project uses survey data (452 respondents, ages 11-19) from a Google Sheet to examine relationships between digital device usage and psychological health outcomes through comprehensive statistical analysis and machine learning.

### Key Research Questions
- How does daily screen time correlate with depression severity (PHQ-9 scores)?
- What demographic, physical, and behavioral factors predict screen usage patterns?
- Which variables show statistically significant associations with mental health outcomes?

## Development Environment Setup

### Prerequisites
- Python 3.11+
- Virtual environment (`env/` directory already configured)
- Jupyter Notebook/Lab

### Initial Setup
1. **Activate virtual environment:**
   ```bash
   # Windows
   env\Scripts\activate
   # macOS/Linux
   source env/bin/activate
   ```

2. **Install dependencies** (if needed):
   ```bash
   pip install -r requirements.txt
   ```

3. **Configure Google Sheets access:**
   - Create/verify `.env` file with `GOOGLE_SHEET_URL` variable
   - URL must point to a publicly shareable Google Sheet
   - File is git-ignored for security

## Project Architecture

### Data Pipeline

The analysis follows a sequential multi-notebook workflow. Each notebook builds on outputs from previous stages:

1. **1_eda.ipynb** (Exploratory Data Analysis & Data Cleaning)
   - Loads data from Google Sheets via `gsheet-loader`
   - Handles encoding issues and character normalization
   - Calculates PHQ-9 depression scores (0-27 scale mapped to 5 severity categories)
   - Creates derived categorical features:
     - Age classes: 11-13, 14-16, 17-19 years
     - BMI classes: Underweight, Normal, Overweight, Obese
     - Income classes: Low (≤150K), Middle (150-300K), High (>300K)
     - Time categories: Bedtime (3 classes), Wake time (3 classes)
   - Outputs `dataset_v2.csv` directly after cleaning and feature engineering
   - Outputs: 452 rows × 38 columns with all raw and engineered features

2. **2_cross_tab_analysis.ipynb** (Feature Consolidation & Category Merging)
   - Reads `dataset_v2.csv`
   - Drops low-value columns: `A4_marital`, `E4_addictive_drug`, `F2_screen_impact`, `F3_learning_interest`
   - Merges underrepresented categories to improve statistical power:
     - Religion: Hindu + Christian → "hindu/christian"
     - Family type: Extended joint → "Joint family"
     - Device: Laptop + Desktop → "Computer/Laptop"
     - BMI: Overweight + Obese → "Overweight"
     - `B6_relationship_with_family`: "Very good" → "Good"
     - `B5_time_with_parents_hours`: regrouped into "Less than 2 hours" / "2-6 hours"
     - `D5_parents_screen_time`: regrouped into "Less than 3 hours" / "3-5 hours"
     - `D2_weekend_screen`: regrouped into fewer bins
     - `C3_physical_problems`: re-bucketed into 4 categories (Headache, Mood-related, Combined, Other)
   - Renames `PHQ_class` value "Moderately depressive" → "Moderately Severe depressive"
   - Creates `D4_content_grouped`: Groups raw content types into 4 categories
     - Educational Content
     - Entertainment & Media (games, videos, music)
     - Social Media & Mixed Usage
     - Other
   - Outputs: `dataset_v3.csv` (452 rows × 35 columns) — the **main cleaned dataset**
   - Note: contains legacy commented-out `analysis()` PDF-generation code (dead, not executed)

3. **3_sig_analysis.ipynb** (Statistical Significance Screening)
   - Performs chi-square tests for all variables vs. two main outcomes:
     - **Target 1: D1_daily_screen** (daily screen time)
     - **Target 2: PHQ_class** (depression severity)
   - Creates filtered datasets keeping only the strictest-significance predictors (`***`, i.e. p < 0.001 — not the general p < 0.05 threshold):
     - `dataset_sig_st.csv`: 20 features significantly associated with daily screen time
     - `dataset_sig_phq9.csv`: 14 features significantly associated with PHQ-9 depression
   - Outputs: Summary chi-square tables with p-values and test statistics

4. **4_mlr_st.ipynb** (Multinomial Logistic Regression — Screen Time)
   - Reads `dataset_sig_st.csv`
   - Predicts daily screen time (4 categories: <1hr, 1-3hrs, 3-5hrs, >5hrs)
   - Produces:
     - Classification metrics (accuracy ~46%, per-class precision/recall)
     - Goodness-of-fit diagnostics (likelihood ratio test, McFadden's pseudo-R² = 0.40, "very good fit")
     - Odds ratio tables with 95% CIs and p-values
     - Feature importance heatmaps
     - Confusion matrices
   - Key outputs: PDF reports in `report_v3/`

5. **5_mlr_phq9.ipynb** (Multinomial Logistic Regression — Depression)
   - Reads `dataset_sig_phq9.csv` (not `dataset_v3.csv`)
   - Predicts depression severity (5 categories: Minimal, Mild, Moderate, Moderately Severe, Severe)
   - Same structure as notebook 4 (odds ratios, GOF diagnostics, confusion matrix), but notably weaker:
     - Accuracy ~37% (vs. ~46% for the screen-time model)
     - Zero precision/recall on the two rarest classes (Moderately Severe, Severe) — the model never predicts them in the test set
     - McFadden's pseudo-R² = 0.118 ("acceptable fit", vs. 0.40 "very good fit" for notebook 4)
   - Key outputs: PDF reports in `report_v3/`

6. **6_analysis_for_mind_well.ipynb** (Expanded Cross-Tabulation Analysis)
   - Reads `dataset_v3.csv`
   - Cross-tabulates 8 "outcome" variables (screen time, weekend screen time, primary device, sleep duration, bedtime, anxious-without-device, feel low/unmotivated, academic satisfaction) against physical-health and mental-health predictor sets
   - Defines its own local `analysis()` PDF-report generator (same pattern as the legacy commented-out versions in notebooks 1/2, but live here)
   - Outputs: 16 PDF reports in `report_v4/` (e.g., `1.Screen_Time_vs_Mental_Health.pdf`, `2.Screen_Time_vs_Physical_Health.pdf`, ... `16.Academic_Satisfaction_vs_Mental_Health.pdf`)

7. **outlier_detection.ipynb** (Data-Quality Diagnostic)
   - Loads data directly from Google Sheets via `gsheet_loader` (independent of the dataset_v2/v3/sig pipeline; duplicates notebook 1's raw-load/encoding-fix cells)
   - Detects outliers using both IQR (1.5×IQR fences) and Z-score (|z| > 3) methods
     - Numeric columns: per-column outlier counts + boxplots
     - Categorical columns: over/under-represented categories treated as outliers, with bar charts highlighting them
   - Outputs: `outlier_numeric.pdf` and `outlier_categorical.pdf` (saved into `report_v4/`)

### Data Files

| File | Source | Purpose | Rows | Cols |
|------|--------|---------|------|------|
| `dataset_v2.csv` | Notebook 1 | Raw cleaned data (intermediate) | 452 | 38 |
| `dataset_v3.csv` | Notebook 2 | Main cleaned dataset (final) | 452 | 35 |
| `dataset_sig_st.csv` | Notebook 3 | Significant predictors of screen time | 452 | 20 |
| `dataset_sig_phq9.csv` | Notebook 3 | Significant predictors of depression | 452 | 14 |

### Report Outputs

Generated analysis reports are organized by version:
- `report_v1/`: Initial cross-tabulation analyses
- `report_v2/`: Refined analyses with subset of variables
- `report_v3/`: MLR odds ratio tables and statistical summaries
- `report_v4/`: Expanded cross-tabulations from notebook 6 (16 PDFs: screen time/device/sleep/bedtime/anxiety/mood/academic satisfaction vs. physical & mental health) + outlier-detection diagnostics from `outlier_detection.ipynb` (`outlier_numeric.pdf`, `outlier_categorical.pdf`)

Reports include:
- Cross-tabulation tables with chi-square test results
- Odds ratio estimates with confidence intervals
- Feature importance visualizations
- Outlier diagnostics (IQR and Z-score based)

## Key Features & Transformations

### PHQ-9 Depression Scoring
- Maps 4-point scale ("Not at all" → 0, "Nearly every day" → 3) to 9 items
- Total score (0-27) categorized as:
  - Minimal: 0-4
  - Mild: 5-9
  - Moderate: 10-14
  - Moderately Severe: 15-19
  - Severe: 20-27

### Feature Engineering Patterns
All categorical variables follow standardized binning to optimize chi-square tests:
- Income: 3 categories (Low/Middle/High)
- Age: 3 categories (11-13, 14-16, 17-19)
- Screen time: 4 categories (ranges in hours)
- Physical symptoms: Grouped by primary symptom type
- Content consumption: 4 grouped categories (see notebook 2)

## Running the Analysis

### Single Notebook Execution
1. Launch Jupyter:
   ```bash
   jupyter notebook
   ```
2. Open desired notebook (1_eda.ipynb through 6_analysis_for_mind_well.ipynb, or outlier_detection.ipynb)
3. Execute cells sequentially — dependencies exist across notebooks

### Common Tasks

**Regenerate entire pipeline (start to finish):**
- Run 1_eda.ipynb → 2_cross_tab_analysis.ipynb → 3_sig_analysis.ipynb
- Then selectively run 4_mlr_st.ipynb, 5_mlr_phq9.ipynb, or 6_analysis_for_mind_well.ipynb
- `outlier_detection.ipynb` can be run independently at any time (loads its own data directly from Google Sheets)

**Update analysis with new data:**
1. Update Google Sheet URL in `.env`
2. Re-run 1_eda.ipynb to import and clean data
3. Continue from notebook 2

**Generate specific regression model:**
- For screen time: Run 4_mlr_st.ipynb (uses dataset_sig_st.csv)
- For depression: Run 5_mlr_phq9.ipynb (uses dataset_sig_phq9.csv)

**Generate PDF cross-tabulation reports:**
- Run analysis() function in notebooks 2/6 with `save_image=True`
- Specify `target` variable and `variables` list
- PDFs saved to `report_v#/` directory

## Key Dependencies

| Package | Purpose |
|---------|---------|
| pandas | Data manipulation and analysis |
| numpy | Numeric operations (notebooks 4, 5, 6, outlier_detection) |
| scikit-learn | Logistic regression, preprocessing (LabelEncoder) |
| scipy.stats | Chi-square tests (chi2_contingency), Z-score outlier detection |
| statsmodels | Advanced logistic regression with p-values/CIs (MNLogit) |
| gsheet-loader | Google Sheets data import |
| python-dotenv | Environment variable management (.env) |
| matplotlib | PDF report generation, boxplots |
| seaborn | Visualization (heatmaps) |

## Important Notes

### Data Handling
- Google Sheet URL stored in `.env` (git-ignored)
- Missing values: None in current cleaned datasets

### Statistical Methodology
- Chi-square tests used for categorical associations; significance tiers reported as `*` (p<0.05), `**` (p<0.01), `***` (p<0.001)
- Notebook 3's filtered datasets (`dataset_sig_st.csv`, `dataset_sig_phq9.csv`) keep only `***` (p < 0.001) predictors — stricter than the general α = 0.05 threshold
- Multinomial logistic regression uses reference class encoding
- Odds ratios > 1 indicate increased likelihood of outcome, < 1 indicates decreased likelihood
- Sample size: 452 respondents (no weighting applied)

### Dataset Versioning
- v2: Immediate output from data cleaning (pre-category merging)
- v3: Final cleaned dataset after category consolidation (used for analyses)
- sig_st: Filtered for screen time analysis only (statistically significant vars)
- sig_phq9: Filtered for depression analysis only (statistically significant vars)

## Debugging & Troubleshooting

**Missing .env file:**
- Create it with: `GOOGLE_SHEET_URL=https://docs.google.com/spreadsheets/d/...`

**Google Sheets import fails:**
- Verify URL is publicly shareable (Sheet is published)
- Check URL format matches example in README

**Chi-square test errors:**
- Expected frequency < 5 in cells may cause warnings (acceptable with large df)
- Consider merging rare categories (already done in notebook 2)

**MLR accuracy seems low (~46% for screen time, ~37% for PHQ-9):**
- This is expected for multinomial classification with 4-5 categories
- The PHQ-9 model (notebook 5) performs notably worse than the screen-time model (notebook 4), with zero precision/recall on the rarest classes (Moderately Severe, Severe depressive) — the minority classes are too small in the test split to be predicted at all
- Focus on odds ratios and class-specific precision/recall, not overall accuracy

