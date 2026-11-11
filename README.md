# Pharmacovigilance Signal Analysis

Exploratory pharmacovigilance analysis of FDA adverse-event reports for three diabetes medications: **Metformin**, **Jardiance (empagliflozin)** and **Ozempic (semaglutide)**. Data is pulled from the [openFDA Drug Adverse Event API](https://open.fda.gov/apis/drug/event/) (FAERS), cleaned into report-level and reaction-level tables, and analysed for reporting patterns, seriousness, and potential safety signals.

**Important:** openFDA data are *spontaneous reports*. The results describe reporting patterns only. They do **not** measure incidence, establish causality, or show that one drug is safer or riskier than another.

## Key results (2014–2018 window)

| Drug | Reports | Reports marked serious | 95% CI (Wilson) |
|---|---:|---:|---|
| Metformin | 136,922 | 63.2% | 62.9 – 63.4% |
| Jardiance | 8,291 | 53.1% | 52.0 – 54.2% |
| Ozempic | 1,175 | 23.0% | 20.7 – 25.5% |

- **Reporting-period bias is large.** Ozempic had only 3–13 reports per year in 2014–2017 and 1,140 in 2018, while Metformin and Jardiance have reports in every year. Only **2018** has more than 30 reports for all three drugs, so the notebook repeats the comparison on that common period.
- Serious-report proportions differ significantly across drugs (chi-square, p < 0.001), but this is a statement about *reporting*, not clinical risk.
- The most frequently reported reaction for Jardiance is diabetic ketoacidosis (about 1,100 reports); for Metformin it is blood glucose increased; for Ozempic it is nausea.
- Drug–reaction pairs with fewer than 30 reports are excluded from signal ranking to avoid unstable percentages.

## What the notebook does

1. **Ingest** – pages through the openFDA REST API month by month, with retry/rate-limit handling and respect for the API's skip limit.
2. **Clean** – flattens nested JSON into one row per report, parses dates, converts types, de-duplicates on (drug, report ID), and profiles missing data.
3. **Analyse**
   - serious-report proportions with Wilson confidence intervals and a chi-square test
   - yearly and monthly reporting trends and year-over-year growth
   - top reactions per drug and seriousness by reaction
   - **common-period analysis** and **minimum sample-size filtering** to reduce bias and small-sample noise
   - observed-vs-expected reaction ratios and a transparent, project-defined *Signal Priority Score* (signal strength 40%, seriousness 35%, frequency 25%, each min-max scaled)
   - sex, age and country breakdowns
4. **SQL** – loads the data into SQLite and runs aggregation and window-function (`RANK() OVER`) queries.
5. **Export** – writes Power BI-ready CSVs to `outputs/`.

## Getting started

```bash
git clone <your-repo-url>
cd <your-repo>
pip install requests pandas numpy matplotlib seaborn scipy statsmodels jupyter
jupyter notebook Pharmacovigilance_Analytics.ipynb
```

An openFDA API key is optional (it raises the rate limit). If you have one, set it as an environment variable instead of putting it in the code:

```bash
export OPENFDA_API_KEY="your_key"        # macOS / Linux
setx OPENFDA_API_KEY "your_key"          # Windows
```

Configuration (drugs, years, minimum-report thresholds) lives in the second code cell. A full 2014–2018 download takes a while, so start with a shorter year range if you only want to test it.

## Tech stack

Python · Pandas · NumPy · Matplotlib · Seaborn · SciPy · statsmodels · SQLite / SQL · REST API (openFDA)

## Limitations

- Spontaneous reporting is subject to under-reporting and reporting bias (media attention, regulatory activity, time on market).
- There is no exposure denominator, so report counts cannot be turned into rates of harm.
- A report can list several drugs; a drug appearing in a report does not mean it caused the reaction.
- The "serious" flag is report-level, so "% serious" for a reaction means the share of reports containing that reaction that were flagged serious.
- Drugs are matched by product name text (`medicinalproduct`), which can include brand and spelling variants and unrelated combinations.
- The Signal Priority Score is a project-specific prioritisation heuristic, not a regulatory method such as PRR, ROR or EBGM.

## Possible extensions

Proportional Reporting Ratio (PRR) / Reporting Odds Ratio (ROR) with confidence intervals, restricting to reports where the drug is the *primary suspect*, adding drugs from the same classes, and a Power BI dashboard on the exported CSVs.

## Data source and licence

Data: openFDA / FDA Adverse Event Reporting System (FAERS). See openFDA's [terms of service](https://open.fda.gov/terms/). Add your preferred licence for the code (e.g. MIT).
