# Mental Health in Tech — Analysis

An exploratory data analysis notebook that examines mental health among technology professionals using the **Open Sourcing Mental Illness (OSMH)** survey data from **2014–2019**.

The notebook explores six core themes — stigma, prevalence & treatment, employer support, career risk, work impact & productivity, and year-over-year trends — and produces a set of publication-quality visualizations for each.

---

## Table of Contents

- [Overview](#overview)
- [Data](#data)
- [Database Connection](#database-connection)
- [Requirements & Installation](#requirements--installation)
- [How to Run](#how-to-run)
- [Analysis Themes](#analysis-themes)
  - [Theme 1 — Stigma Gap](#theme-1--stigma-gap)
  - [Theme 2 — Prevalence & Treatment](#theme-2--prevalence--treatment)
  - [Theme 3 — Employer Support](#theme-3--employer-support)
  - [Theme 4 — Career Risk](#theme-4--career-risk)
  - [Theme 5 — Work Impact & Productivity](#theme-5--work-impact--productivity)
  - [Theme 6 — Year-over-Year Trends](#theme-6--year-over-year-trends)
- [Data Quality](#data-quality)
- [Key Findings](#key-findings)
- [Limitations & Notes](#limitations--notes)

---

## Overview

Mental health is a growing concern across the tech industry. This analysis answers questions such as:

- Are people less willing to discuss mental health than physical health?
- How common is mental health treatment-seeking, and how has it changed over time?
- How much support do employers actually provide?
- Do workers fear career consequences more than they observe them happening to others?
- How does a mental health condition affect work productivity, with and without treatment?

All answers are derived programmatically from the raw survey data via SQL queries, so every number is reproducible from source.

---

## Data

- **Source:** Open Sourcing Mental Illness (OSMH) survey
- **Years:** 2014, 2016, 2017, 2018, 2019 (no 2015 survey)
- **Scale:** ~4218 unique respondents, **236,898 answers** across **105 questions**
- **Format:** stored in a relational database (see below), not flat files

### Schema

The data lives in three tables:

| Table | Columns | Rows | Purpose |
|-------|---------|------|---------|
| `Survey` | `SurveyID`, `Description` | 5 | One row per survey year |
| `Question` | `QuestionID`, `QuestionText` | 105 | The survey questions |
| `Answer` | `AnswerID`, `AnswerText`, `SurveyID`, `UserID`, `QuestionID` | 236,898 | Respondent answers |

The `Answer` table is a classic **entity–attribute–value** (EAV) structure: each row ties one user's answer to a question within a specific survey year.

---

## Database Connection

This project **does not** read from CSV files. Instead, the notebook connects directly to a **SQL Server** relational database and pulls the data into pandas DataFrames on demand.

### Connection type

- **Driver:** ODBC Driver 17 for SQL Server
- **Authentication:** Windows Integrated (Trusted) Authentication — `Trusted_Connection=yes`
- **Server / instance:** a local SQL Server Express instance
- **Database:** the schema holds the survey, question, and answer tables

Because Windows Integrated Authentication is used, no username or password is embedded in the connection string — the notebook authenticates using the current Windows user's credentials.

### Connection process

1. The notebook establishes a connection using `pyodbc.connect(...)`, passing a connection string that specifies the driver, the SQL Server instance, the target database, and the trusted-connection flag.
2. Once connected, tables are loaded into `pandas` DataFrames with `pd.read_sql()`.
3. Raw T-SQL queries — including **window functions** such as `OVER()` to compute row-level percentages — are executed directly against the database to build, aggregate, and pivot each analysis theme's dataset.
4. Each dataset is then visualized with `matplotlib` / `seaborn`.
5. At the end of the notebook the connection is explicitly closed.

### ⚠️ Environment-specific configuration

The connection string is **machine-specific**. The server name, SQL instance, and database name will differ on other machines. To run this notebook on your own environment, locate the `conn = pyodbc.connect(...)` block at the top of the notebook and update:

- the **server / instance** name, and
- the **database** name

to match your local SQL Server setup (and confirm the ODBC driver version is installed).

---

## Requirements & Installation

The notebook requires Python 3 with the following packages:

- `pandas`
- `pyodbc`
- `matplotlib`
- `seaborn`

Install with:

```bash
pip install pandas pyodbc matplotlib seaborn
```

### Additional prerequisite

- **Microsoft ODBC Driver 17 for SQL Server** must be installed on the machine, and a reachable SQL Server instance containing the `Survey` / `Question` / `Answer` tables (and matching connection details) must be available.

---

## How to Run

1. Ensure your SQL Server instance is running and that the connection string at the top of the notebook matches your environment (see [Database Connection](#database-connection)).
2. Open the notebook in Jupyter (or your preferred notebook environment):

   ```bash
   jupyter notebook Mental_Health_Analysis.ipynb
   ```

3. Run the cells **top to bottom**.
4. Charts render inline as each theme's analysis completes.

---

## Analysis Themes

### Theme 1 — Stigma Gap

*Would you bring up mental vs physical health in a job interview? Also, how comfortable are people discussing mental health with coworkers vs supervisors?*

Users are far more reluctant to discuss **mental** health than **physical** health during an interview:

| Topic | Yes | Maybe | No |
|-------|-----|-------|----|
| Mental health | 5.5% (231) | 24.6% (1036) | **70.0% (2951)** |
| Physical health | 21.4% (901) | 43.2% (1823) | 35.4% (1494) |

A large majority of respondents (70%) would **not** raise mental health in an interview, while most would be at least somewhat willing to raise physical health. The notebook also compares comfort levels discussing mental health with **coworkers** vs **supervisors**.

- **Key takeaway:** a clear stigma gap exists — mental health is treated as far more sensitive than physical health in a hiring/interview context.

---

### Theme 2 — Prevalence & Treatment

*How many people sought treatment? How many report a current disorder?*

**Treatment-seeking rate by year** (% who sought treatment):

| Survey | PctYes | N |
|--------|--------|---|
| 2014 | 50.6% | 1260 |
| 2016 | 58.5% | 1433 |
| 2017 | 60.3% | 756 |
| 2018 | **63.1%** | 417 |
| 2019 | 61.6% | 352 |

**Current mental health disorder** (all years combined, N = 2958):

- **Yes:** 1237
- **No:** 969
- **Maybe / Possibly:** 628

- **Key takeaway:** treatment-seeking grew from ~51% in 2014 to ~62–63% by 2018–2019, while a substantial share of respondents still report a current (or possible) mental health condition.

---

### Theme 3 — Employer Support

*Benefits, anonymity, wellness campaigns, resources, and whether mental health is treated with the same seriousness as physical health.*

Respondent counts per question (before cleaning):

| Question | Yes | No | Don't know / I don't know |
|----------|-----|----|---------------------------|
| MH benefits offered | 1744 | 756 | 1066 |
| Anonymity protected | 1135 | 213 | 2366 |
| Wellness campaigns | 599 | 1626 | 229 |
| MH resources provided | 702 | 1084 | 668 |
| MH = physical seriousness | 693 | 644 | 1069 |

The notebook cleans text variants into a common scale and plots **stacked horizontal bar charts** of the percentage distribution for each question.

- **Key takeaway:** a large share of respondents either **don't know** whether employer support exists (especially regarding anonymity and benefits) or report it **not** being provided, suggesting both gaps in coverage and gaps in communication.

---

### Theme 4 — Career Risk

*Fear of negative career consequences vs. what respondents have actually observed happening to others.*

| Source | Yes | No | Maybe |
|--------|-----|----|-------|
| Fear (personal) | 21.4% (514) | 38.6% (928) | 40.1% (964) |
| Observed in others | 10.5% (283) | 78.8% (2123) | — |

- **Key takeaway:** roughly 21% of respondents personally fear negative consequences from discussing mental health, but only about **10.5%** have actually observed such consequences happening to others — a gap between perceived and realized career risk.

---

### Theme 5 — Work Impact & Productivity

*How much does a mental health condition interfere with work, treated vs untreated? What share of work time is affected?*

**Interference when treated vs untreated** (respondent counts):

| Frequency | When treated (N) | When untreated (N) |
|-----------|------------------|--------------------|
| Never | 165 | 24 |
| Rarely | 700 | 113 |
| Sometimes | 808 | 672 |
| Often | 166 | **1183** |

**Share of work time affected** (respondent counts):

| % of work time | N |
|----------------|---|
| 1–25% | 164 |
| 26–50% | 125 |
| 51–75% | 53 |
| 76–100% | 25 |

- **Key takeaway:** untreated conditions cause far more frequent work interference — "Often" jumps from 166 (treated) to **1183** (untreated) — while treatment shifts responses toward "Rarely"/"Never". Most affected workers lose up to half of their work time.

---

### Theme 6 — Year-over-Year Trends

*Key metrics tracked across all five survey years.*

The notebook plots five metrics in a single multi-line chart over 2014–2019:

- **Treatment-seeking** (%)
- **Discuss in interview** (% who would)
- **Comfort with supervisor** (%)
- **Benefits offered** (%)
- **Current disorder** (%)

Values by year (%):

| Year | TreatmentSeeking | CurrentDisorder | InterviewMH_Yes | ComfortSupervisor | BenefitsYes |
|------|------------------|-----------------|-----------------|-------------------|-------------|
| 2014 | 50.6 | — | 3.5 | 41.0 | 37.9 |
| 2016 | 58.5 | 40.1 | 7.8 | 29.9 | 37.1 |
| 2017 | 60.3 | 42.9 | 5.0 | 33.9 | 47.5 |
| 2018 | 63.1 | 45.8 | 4.8 | 31.4 | 51.1 |
| 2019 | 61.6 | 41.8 | 4.8 | 33.2 | 46.6 |

*Note: the current-disorder metric is absent for 2014 and the interview/disorder questions were not asked consistently in every year (some cells are `NaN`).*

- **Key takeaway:** treatment-seeking and employer-provided benefits both trend upward over time, while self-reported current disorder hovers around 40–46% and willingness to discuss mental health in an interview remains very low (~5%).

---

## Data Quality

The dataset is clean out of the box:

- **No missing values** in any of the three tables.
- **No duplicate rows** in any of the three tables.

As a result, **no preprocessing** was required before analysis. The only light cleanup performed in-notebook is standardizing textual answer variants (e.g., `"I don't know"` vs `"Don't know"`) and excluding placeholder values such as `-1`.

---

## Key Findings

1. **Stigma is real:** only 5.5% of respondents would raise mental health in a job interview, versus 21.4% for physical health; 70% would not.
2. **Treatment-seeking is rising:** from ~51% (2014) to ~62–63% (2018/2019).
3. **Disorder prevalence is high and stable:** ~40–46% of respondents report a current (or possible) mental health condition.
4. **Employer support is uneven:** large "don't know" shares suggest both coverage and communication gaps (e.g., anonymity — ~2,366 don't know).
5. **Fear exceeds observed harm:** ~21% fear career consequences, but only ~10.5% have observed them.
6. **Treatment materially helps productivity:** work interference "Often" drops dramatically from 1183 (untreated) to 166 (treated).
7. **Slow culture shift:** willingness to discuss mental health in interviews remains very low (~5%) even as treatment-seeking and benefits improve.

---

## Limitations & Notes

- **Connection is environment-specific:** the pyodbc connection string (server/instance, database, ODBC driver) must be adjusted to run on another machine. See [Database Connection](#database-connection).
- **No 2015 data:** the OSMH survey was not conducted / included for 2015, creating a gap in the time series.
- **Inconsistent questions across years:** several themes rely on question IDs that were not all asked every year, so some trend cells are missing (`NaN`), and some analyses use different but equivalent question IDs grouped together (e.g., `98,104`).
- **Placeholder / free-text answers:** values such as `-1` represent non-responses and are filtered during analysis; text variants are normalized in-notebook.
