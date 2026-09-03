# 0.1) Example Data Science Project

---

- Last Update: 2026-08-01
- Source: [00_01_introduction_example_project.md](/learning-modules/intro-ds-module/00_01_introduction_example_project.md)

---

## Outline

- [Outline](#outline)
- [Overall philosophy](#overall-philosophy)
- [Learning objectives](#learning-objectives)
- [Module structure](#module-structure)
  - [Week 0 – Welcome](#week-0--welcome)
  - [Week 1 – Data Science Project Environment](#week-1--data-science-project-environment)
  - [Week 2 – Data Management](#week-2--data-management)
  - [Week 3 – Data Preparation \& Visualization](#week-3--data-preparation--visualization)
  - [Week 4 – Data Analysis](#week-4--data-analysis)
  - [Week 5 – Summary](#week-5--summary)
- [Running project repository](#running-project-repository)
- [Data sources](#data-sources)

---

## Overall philosophy

The module is built around **one continuous, reproducible data science project** rather than a collection of disconnected examples.

**Running case study**:

❗**Understanding changes in maize yield in Southern Africa**

This allows every topic to be introduced within the same workflow:

```text
Research Question
        ↓
Data Acquisition
        ↓
Data Management
        ↓
Data Preparation
        ↓
Exploratory Data Analysis
        ↓
Visualization
        ↓
Modeling
        ↓
Evaluation
        ↓
Interpretation
        ↓
Communication & Reproducibility
```

The project should use real **FAOSTAT** data and, optionally, selected World Bank indicators.

---

## Learning objectives

After completing the module, participants should be able to

- organize a reproducible data science project;
- understand common food-system datasets;
- manage and document data;
- clean and prepare data for analysis;
- create informative visualizations;
- fit and interpret simple statistical models;
- distinguish explanation from prediction;
- communicate results in a reproducible report.

---

## Module structure

### Week 0 – Welcome

- Introduction
- Expectations
- Course organization
- Overview of the complete workflow

---

### Week 1 – Data Science Project Environment

**Topics**

- Project organization
- RStudio Project
- Git
- GitHub / GitLab
- Reproducible environments
- renv
- Containers (overview)
- Local vs remote computing
- SSH (conceptual)

**Deliverable**

- Students can run the supplied project and reproduce all analyses.

**Suggested resources**

- The Turing Way (Reproducibility)

---

### Week 2 – Data Management

**Topics**

- What is data?
- Structured vs semi-structured data
- Cross-sectional
- Time series
- Panel
- Spatial data
- CSV
- Excel
- JSON
- Databases
- Metadata
- FAIR principles
- Data provenance

**Running example**

- Import FAOSTAT maize production data.

**Students inspect**

- variables
- units
- countries
- years
- missing values
- metadata

**Deliverable**

- A documented data dictionary.

---

### Week 3 – Data Preparation & Visualization

**Topics**

Data preparation

- filtering
- selecting variables
- joins
- missing values
- duplicates
- reshaping
- derived variables

Visualization

- distributions
- time series
- comparisons
- scatterplots

**Running example**

Questions

- Which countries have the largest maize yields?
- How have yields changed over time?
- Are changes driven by production or harvested area?

**Deliverable**

- A reproducible Quarto/HTML report with visualizations.

---

### Week 4 – Data Analysis

**Topics**

Goals of analysis

- explanation
- prediction

Model workflow

- train/test split
- generalization
- evaluation
- interpretation

Models

- linear regression
- multiple regression

**Running example**

Example descriptive model

```r
lm(log(yield) ~ year + country)
```

Prediction exercise

- Training: 1990–2017
- Testing: 2018+

Students compare

- historical mean
- linear trend
- country model

**Deliverable**

- Interpret model coefficients and prediction performance.

---

### Week 5 – Summary

Students submit a short reproducible report including

- research question
- data
- preparation
- visualizations
- model
- interpretation
- limitations
- reproducibility statement

---

## Running project repository

```text
maize-yield-project/
├── README.md
├── data-raw/
├── data-processed/
├── scripts/
├── reports/
├── figures/
├── renv.lock
└── maize-yield-project.Rproj
```

---

## Data sources

**Primary**

- FAOSTAT

**Optional**

- World Bank Open Data

**Possible variables**

- maize production
- harvested area
- yield
- fertilizer use
- agricultural land
