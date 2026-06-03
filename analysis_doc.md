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
   - Merges underrepresented categories to improve statistical power:
     - Religion: Hindu + Christian → "hindu/christian"
     - Family type: Extended joint → "Joint family"
     - Device: Laptop + Desktop → "Computer/Laptop"
     - BMI: Overweight + Obese → "Overweight"
   - Creates `D4_content_grouped`: Groups raw content types into 4 categories
     - Educational Content
     - Entertainment & Media (games, videos, music)
     - Social Media & Mixed Usage
     - Other
   - Outputs: `dataset_v3.csv` (452 rows × 35 columns) — the **main cleaned dataset**

3. **3_sig_analysis.ipynb** (Statistical Significance Screening)
   - Performs chi-square tests for all variables vs. two main outcomes:
     - **Target 1: D1_daily_screen** (daily screen time)
     - **Target 2: PHQ_class** (depression severity)
   - Creates filtered datasets with only statistically significant predictors (p < 0.05):
     - `dataset_sig_st.csv`: 20 features significantly associated with daily screen time
     - `dataset_sig_phq9.csv`: 14 features significantly associated with PHQ-9 depression
   - Outputs: Summary chi-square tables with p-values and test statistics

4. **4_mlr_st.ipynb** (Multinomial Logistic Regression — Screen Time)
   - Reads `dataset_sig_st.csv`
   - Predicts daily screen time (4 categories: <1hr, 1-3hrs, 3-5hrs, >5hrs)
   - Produces:
     - Classification metrics (accuracy ~46%, per-class precision/recall)
     - Odds ratio tables with 95% CIs and p-values
     - Feature importance heatmaps
     - Confusion matrices
   - Key outputs: PDF reports in `report_v3/`

5. **5_mlr_phq9.ipynb** (Multinomial Logistic Regression — Depression)
   - Reads `dataset_v3.csv`
   - Predicts depression severity (5 categories: Minimal, Mild, Moderate, Moderately Severe, Severe)
   - Similar structure to notebook 4 with depression-specific insights

6. **6_analysis_for_mind_well.ipynb** (Custom Analysis)
   - Latest analysis notebook (minimal code visible)
   - Purpose: TBD based on ongoing research needs

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

Reports include:
- Cross-tabulation tables with chi-square test results
- Odds ratio estimates with confidence intervals
- Feature importance visualizations

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
2. Open desired notebook (1_eda.ipynb through 6_analysis_for_mind_well.ipynb)
3. Execute cells sequentially — dependencies exist across notebooks

### Common Tasks

**Regenerate entire pipeline (start to finish):**
- Run 1_eda.ipynb → 2_cross_tab_analysis.ipynb → 3_sig_analysis.ipynb
- Then selectively run 4_mlr_st.ipynb or 5_mlr_phq9.ipynb

**Update analysis with new data:**
1. Update Google Sheet URL in `.env`
2. Re-run 1_eda.ipynb to import and clean data
3. Continue from notebook 2

**Generate specific regression model:**
- For screen time: Run 4_mlr_st.ipynb (uses dataset_sig_st.csv)
- For depression: Run 5_mlr_phq9.ipynb (uses dataset_v3.csv or filtered version)

**Generate PDF cross-tabulation reports:**
- Run analysis() function in notebooks 3/4/5 with `save_image=True`
- Specify `target` variable and `variables` list
- PDFs saved to `report_v#/` directory

## Key Dependencies

| Package | Purpose |
|---------|---------|
| pandas | Data manipulation and analysis |
| scikit-learn | Logistic regression, preprocessing (LabelEncoder) |
| scipy.stats | Chi-square tests (chi2_contingency) |
| statsmodels | Advanced logistic regression with p-values/CIs |
| gsheet-loader | Google Sheets data import |
| python-dotenv | Environment variable management (.env) |
| matplotlib | PDF report generation |
| seaborn | Visualization (heatmaps) |

## Important Notes

### Data Handling
- Google Sheet URL stored in `.env` (git-ignored)
- Missing values: None in current cleaned datasets

### Statistical Methodology
- Chi-square tests used for categorical associations (α = 0.05)
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

**MLR accuracy seems low (~46%):**
- This is expected for multinomial classification with 4+ categories
- Focus on odds ratios and class-specific precision/recall, not overall accuracy

