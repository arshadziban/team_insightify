<div align="center">
  <img src="logo.svg" alt="Insightify Logo" width="200" />
  
  **Analyzing Screen Time Impact on Mental Health & Well-being**
  
  A comprehensive data analysis project examining the relationship between digital device usage and psychological health outcomes in adolescents.
</div>

## Team Members

- **Nishat Tasnim**
- **Ayesha Muni**

*Final Year Design Project*

## Project Overview

Insightify is a data-driven research initiative that investigates the multifaceted impact of screen time on mental health and overall well-being. Through rigorous statistical analysis and data visualization, this project provides insights into behavioral patterns, depression indicators, sleep quality, and physical health metrics in relation to digital media consumption.

### Key Objectives

- Analyze correlations between daily screen time and mental health indicators
- Categorize depression severity levels using PHQ-9 assessment scores
- Evaluate sleep patterns and their relationship with device usage
- Examine demographic and socioeconomic factors influencing screen habits
- Generate actionable insights for health interventions

## Features

**Data Cleaning & Preprocessing**
- Character encoding normalization
- Categorical binning and classification
- Time format conversion (24-hour to 12-hour)

**Comprehensive Metrics Calculation**
- BMI classification (Underweight, Normal, Overweight, Obese)
- PHQ-9 depression severity scoring
- Income-based socioeconomic categorization
- Sleep duration and bedtime classification

**Statistical Analysis**
- Cross-tabulation analysis
- Chi-square tests for association
- Demographic stratification

**Data Visualization**
- Income distribution charts
- Mental health trend analysis
- Demographic breakdowns

## Dataset

The project analyzes survey data from Google Sheets containing:

- **Demographics:** Age, gender, religion, marital status, residence
- **Family Metrics:** Income, family type, time with parents, family relationships
- **Physical Health:** Height, weight, BMI, physical problems
- **Screen Usage:** Daily screen time, weekend usage, device type, content preferences, parental screen time
- **Mental Health:** PHQ-9 depression scores, anxiety, mood patterns, communication issues
- **Sleep Quality:** Sleep duration, bedtime, wake time, sleep medication
- **Academic Impact:** Academic satisfaction, learning interest, screen impact on studies
- **Behavioral Patterns:** Mobile phone habits, appetite changes, AI usage

## Getting Started

### Prerequisites

- Python 3.11+
- Virtual environment (recommended)

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/arshadziban/team_insightify.git
cd team_insightify
```

2. **Create and activate virtual environment**
```bash
python -m venv env
# On Windows:
env\Scripts\activate
# On macOS/Linux:
source env/bin/activate
```

3. **Install dependencies**
```bash
pip install -r requirements.txt
```

4. **Configure environment variables**
Create a `.env` file in the project root:
```
GOOGLE_SHEET_URL=your_secure_google_sheet_url_here
```
*Store your actual Google Sheets sharing link securely in this file. This file is automatically excluded from version control.*

### Running the Analysis

1. **Open the Jupyter Notebook**
```bash
jupyter notebook thesis_25.ipynb
```

2. **Execute cells sequentially** to:
   - Load data from Google Sheets (Cell 1)
   - Perform data cleaning and encoding fixes (Cells 2-5)
   - Calculate PHQ-9 depression scores (Cells 6-10)
   - Create demographic categories: age, BMI, income (Cells 11-20)
   - Refine data features: height, time, BMI, income (Cells 21-28)
   - Categorize physical problems and digital content (Cells 29-33)
   - Time-based categorization: bedtime, wake time (Cells 34-38)
   - Create processed dataset (Cells 39-40)
   - Generate initial analysis report (Cell 41)

3. **Optional: Activate detailed statistical analysis**
   - Uncomment Cells 45-46 for chi-square tests and PDF report generation
   - Requires adjusting the target variable and independent variables as needed

## Project Structure

```
team_insightify/
├── README.md                      # Project documentation
├── .env                           # Environment variables (not tracked)
├── .gitignore                     # Git ignore rules
├── requirements.txt               # Python dependencies
├── logo.svg                       # Insightify project logo
├── thesis_25.ipynb               # Main analysis notebook
├── d1_daily_screen_report.html   # Generated analysis report (D1 daily screen time analysis)
└── report/                        # Generated reports directory
```

## Analysis Workflow

### Phase 1: Data Acquisition & Cleaning
- Load survey data from Google Sheets
- Handle encoding issues and normalize text
- Remove irrelevant columns

### Phase 2: Feature Engineering
```python
# Depression Classification (PHQ-9)
PHQ_class → [Minimal, Mild, Moderate, Moderately Severe, Severe]

# Age Categorization
age_class → [11-13 years, 14-16 years, 17-19 years]

# BMI Classification
BMI_class → [Underweight, Normal, Overweight, Obese]

# Income Stratification
income_class → [Low (≤150K), Middle (150-300K), High (>300K)]
```

### Phase 3: Exploratory Analysis
- Distribution analysis of key variables
- Cross-tabulation of screen time vs. mental health
- Demographic segmentation

### Phase 4: Statistical Testing (Optional)
- Chi-square tests for categorical associations (currently commented out in notebook)
- Significance testing (p < 0.05)
- Multiple comparison analysis
- Can be activated by uncommenting relevant code cells

## Key Findings

The analysis reveals important relationships between:
- **Screen time patterns** and depression severity levels
- **Sleep quality** and device usage habits
- **Socioeconomic factors** and digital consumption
- **Academic performance** and screen dependency

## Output Reports

- **d1_daily_screen_report.html** - Interactive analysis report generated using the deep-study library, comparing daily screen time across demographic groups and providing statistical insights
- **report/** - Directory containing additional generated analysis reports and outputs

*Note: Dataset export and detailed statistical analysis tables are commented out in the notebook and can be activated as needed.*

## Technologies Used

| Technology | Purpose |
|-----------|---------|
| **Python 3.11** | Data processing & analysis |
| **Pandas** | Data manipulation & analysis |
| **Scikit-learn** | Statistical testing & ML preprocessing |
| **Gsheet-loader** | Google Sheets data integration |
| **Jupyter** | Interactive analysis environment |
| **Python-dotenv** | Environment variable management |

## Dependencies

All project dependencies are listed in `requirements.txt`:
- pandas
- requests
- scikit-learn
- gsheet-loader
- python-dotenv

*Additional Libraries Used (installed via dependencies):*
- numpy (via pandas/scikit-learn)
- scipy (via scikit-learn)

## Security & Privacy

- `.env` file is git-ignored to protect sensitive URLs
- All personal identifiable information (PII) from surveys is anonymized
- Data is processed locally and securely
- No sensitive data is committed to version control

## Documentation

For detailed methodology and interpretation of results, refer to:
- Comments within `thesis_25.ipynb`
- Generated HTML reports in the project root
- Statistical test documentation and assumption checks

## Contributing

This is a final year academic project. For contributions or questions:
1. Contact team members directly
2. Review existing analysis code
3. Propose methodological improvements via discussion

## License

This project is for academic purposes as part of a university final year design project.

## Contact & Support

**Team Insightify**
- Nishat Tasnim
- Ayesha Muni

For questions about the analysis or methodology, please reach out to the team members.

<div align="center">
  <strong>Insightify:</strong> Understanding the Digital Age Impact on Well-being
  
  *Final Year Design Project*
</div>
