# 8.3) Describe maize yield and precipitation

---

- Last Update: 2026-08-25
- Source: [08_03_descriptive_analysis_application.md](/learning-modules/intro-ds-module/08_03_descriptive_analysis_application.md)
- Estimated completion time: 6–8 hours
- Independent extension: 2–3 hours
- Prerequisites: Motivation and Concepts pages; completed Data Visualization workflow
- Required output: descriptive-analysis script, summary tables, stationarity diagnostic, and modeling handoff

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Scenario and deliverables](#scenario-and-deliverables)
- [Before you begin](#before-you-begin)
- [1. Define descriptive contracts](#1-define-descriptive-contracts)
- [2. Inspect coverage and denominators](#2-inspect-coverage-and-denominators)
- [3. Describe maize yield by country](#3-describe-maize-yield-by-country)
- [4. Compare meaningful periods](#4-compare-meaningful-periods)
- [5. Describe precipitation](#5-describe-precipitation)
- [6. Quantify yield-precipitation association](#6-quantify-yield-precipitation-association)
- [7. Assess evidence about stability](#7-assess-evidence-about-stability)
- [8. Write the modeling handoff](#8-write-the-modeling-handoff)
- [Independent extension](#independent-extension)
  - [Option A: Within-country deviations](#option-a-within-country-deviations)
  - [Option B: Rolling descriptions](#option-b-rolling-descriptions)
  - [Option C: Lag-one autocorrelation](#option-c-lag-one-autocorrelation)
  - [Option D: Period sensitivity](#option-d-period-sensitivity)
- [Troubleshooting](#troubleshooting)
- [Completion checklist](#completion-checklist)
- [Reflect on the application](#reflect-on-the-application)
- [Further resources](#further-resources)

---

## Learning objectives

After completing this exercise, you should be able to:

- define the population, grain, groups, periods, missing-value rules, and measures for a descriptive analysis;
- calculate reproducible coverage, location, dispersion, and quantile summaries with R;
- compare pooled, country-specific, and period-specific results;
- calculate covariance and Pearson correlation with paired observation counts;
- inspect trends, changing variation, and association drift as evidence relevant to stationarity;
- distinguish descriptive association from causal or predictive claims; and
- translate descriptive findings into requirements for later modeling and evaluation.

---

## Place in the session

This is the **Application** part of the Descriptive Data Analysis session:

~~~text
Motivation  →  Concepts  →  Application
                              ↑
                           this page
~~~

Before beginning, review [Why describe data
numerically?](08_01_descriptive_analysis_motivation.md) and [Understand
descriptive-data-analysis concepts](08_02_descriptive_analysis_concepts.md).

The preceding visualization topic created reproducible figures from prepared
and integrated data. This exercise quantifies selected visual patterns. It
does not alter the input datasets or fit an explanatory or predictive model.

---

## Scenario and deliverables

The project team must answer:

> How do maize yield and growing-season precipitation vary across countries
> and periods, how stable do their distributions and association appear, and
> what must later models represent?

Create these artifacts:

~~~text
scripts/describe-maize-data.R
results/tables/descriptive-coverage.csv
results/tables/maize-yield-summary.csv
results/tables/maize-yield-period-summary.csv
results/tables/precipitation-summary.csv
results/tables/yield-precipitation-association.csv
results/tables/stationarity-diagnostic.csv
results/descriptive-modeling-handoff.md
~~~

The tables provide machine-readable evidence. The Markdown handoff explains
what that evidence means, what it does not establish, and how it affects later
sessions.

---

## Before you begin

Work from the standalone `maize-yield-project` repository. Check your location,
branch, and working tree:

~~~bash
pwd
git status --short --branch
~~~

Restore the environment and recreate the offline inputs:

~~~bash
Rscript scripts/setup.R
Rscript scripts/validate-data.R
Rscript scripts/prepare-maize-data.R
Rscript scripts/integrate-data.R
Rscript scripts/visualize-maize-data.R
~~~

Review these files:

| Role | File |
| --- | --- |
| Integrated data | `data/derived/maize-yield-with-precipitation.csv` |
| Data documentation | `docs/data/maize-yield-with-precipitation.md` |
| Data dictionary | `metadata/maize-yield-with-precipitation-data-dictionary.csv` |
| Preparation and integration evidence | `results/tables/data-preparation-audit.csv`, `results/tables/data-integration-audit.csv` |
| Visualization evidence | `results/tables/data-visualization-manifest.csv` and `figures/` |

The expected input has 297 unique country-year rows: nine countries observed
annually from 1990 through 2022. Confirm that upstream audits pass.

Start `scripts/describe-maize-data.R` with explicit setup and checks:

~~~r
# Create reproducible descriptive summaries of maize data.

source("scripts/functions.R")
assert_project_root()
ensure_project_directories()
check_required_packages(c("dplyr", "here", "readr", "tidyr"))

library(dplyr)
library(here)
library(readr)
library(tidyr)

input_file <- here(
  "data", "derived", "maize-yield-with-precipitation.csv"
)
maize <- read_csv(input_file, show_col_types = FALSE)

required_columns <- c(
  "project_country_id", "project_country_name", "year",
  "yield_tonnes_per_hectare", "growing_season_precipitation_mm"
)
missing_columns <- setdiff(required_columns, names(maize))

if (length(missing_columns) > 0) {
  stop("Missing column(s): ", paste(missing_columns, collapse = ", "))
}
if (nrow(maize) != 297L ||
    anyDuplicated(maize[c("project_country_id", "year")])) {
  stop("Expected 297 unique project-country-year rows.")
}

maize <- maize |>
  mutate(
    analysis_period = case_when(
      year <= 2005 ~ "1990-2005: earlier history",
      year <= 2017 ~ "2006-2017: recent training",
      TRUE ~ "2018-2022: later test"
    )
  )
~~~

These periods follow the existing modeling workflow. Do not silently change
them after seeing the results.

---

## 1. Define descriptive contracts

Before coding, complete this table:

| Output | Population and grain | Group or period | Measures | Missing-value rule | Claim boundary |
| --- | --- | --- | --- | --- | --- |
| Coverage | Nine project countries; one country-year | Country and full period | Rows, available values, first/last year | Count explicitly | Does not establish representativeness |
| Yield summary | Define this | Define this | Define this | Define this | Define this |
| Period summary | Define this | Define this | Define this | Define this | Define this |
| Precipitation summary | Define this | Define this | Define this | Define this | Define this |
| Association | Paired yield and precipitation | Define this | Covariance, correlation, paired n | Complete pairs | Not causal |
| Stability diagnostic | Define this | Three fixed periods | Define this | Define this | Evidence, not proof |

State units and explain why selected measures complement one another. Do not
add measures merely because software makes them easy to calculate.

---

## 2. Inspect coverage and denominators

Create one row per country:

~~~r
coverage <- maize |>
  group_by(project_country_id, project_country_name) |>
  summarise(
    total_rows = n(),
    first_year = min(year),
    last_year = max(year),
    distinct_years = n_distinct(year),
    non_missing_yield = sum(!is.na(yield_tonnes_per_hectare)),
    missing_yield = sum(is.na(yield_tonnes_per_hectare)),
    non_missing_precipitation =
      sum(!is.na(growing_season_precipitation_mm)),
    missing_precipitation =
      sum(is.na(growing_season_precipitation_mm)),
    .groups = "drop"
  )

write_csv(
  coverage,
  here("results", "tables", "descriptive-coverage.csv"),
  na = ""
)
~~~

Ask whether every country contributes the same years and whether yield and
precipitation are available for the same rows. Equal country-year coverage
does not imply coverage of farms or within-country conditions.

---

## 3. Describe maize yield by country

Create complementary summaries:

~~~r
yield_summary <- maize |>
  group_by(project_country_id, project_country_name) |>
  summarise(
    n = sum(!is.na(yield_tonnes_per_hectare)),
    mean_t_per_ha = mean(yield_tonnes_per_hectare, na.rm = TRUE),
    median_t_per_ha = median(yield_tonnes_per_hectare, na.rm = TRUE),
    sd_t_per_ha = sd(yield_tonnes_per_hectare, na.rm = TRUE),
    q25_t_per_ha = quantile(yield_tonnes_per_hectare, 0.25, na.rm = TRUE),
    q75_t_per_ha = quantile(yield_tonnes_per_hectare, 0.75, na.rm = TRUE),
    iqr_t_per_ha = IQR(yield_tonnes_per_hectare, na.rm = TRUE),
    minimum_t_per_ha = min(yield_tonnes_per_hectare, na.rm = TRUE),
    maximum_t_per_ha = max(yield_tonnes_per_hectare, na.rm = TRUE),
    .groups = "drop"
  )

write_csv(
  yield_summary,
  here("results", "tables", "maize-yield-summary.csv"),
  na = ""
)
~~~

Compare each row with the corresponding time-series panel: where mean and
median differ, where SD and IQR tell different stories, and whether a wide
range reflects persistent variation or one unusual year. These are national
country-year descriptions, not farm-level summaries.

---

## 4. Compare meaningful periods

Calculate the same core measures by country and predefined period:

~~~r
yield_period_summary <- maize |>
  group_by(project_country_id, project_country_name, analysis_period) |>
  summarise(
    first_year = min(year),
    last_year = max(year),
    n = sum(!is.na(yield_tonnes_per_hectare)),
    mean_t_per_ha = mean(yield_tonnes_per_hectare, na.rm = TRUE),
    median_t_per_ha = median(yield_tonnes_per_hectare, na.rm = TRUE),
    sd_t_per_ha = sd(yield_tonnes_per_hectare, na.rm = TRUE),
    iqr_t_per_ha = IQR(yield_tonnes_per_hectare, na.rm = TRUE),
    .groups = "drop"
  )

write_csv(
  yield_period_summary,
  here("results", "tables", "maize-yield-period-summary.csv"),
  na = ""
)
~~~

Compare early with recent training history, then recent training with the
later test period; the test period has only five annual observations per
country, so its SD and IQR are sensitive to individual years. Ask whether
typical yield shifts in the same direction across countries, whether
dispersion changes with location, and whether recent training history
resembles the later evaluation period. These comparisons provide evidence
about stability; they do not prove stationarity.

---

## 5. Describe precipitation

Use the same reporting discipline:

~~~r
precipitation_summary <- maize |>
  group_by(project_country_id, project_country_name, analysis_period) |>
  summarise(
    n = sum(!is.na(growing_season_precipitation_mm)),
    mean_mm = mean(growing_season_precipitation_mm, na.rm = TRUE),
    median_mm = median(growing_season_precipitation_mm, na.rm = TRUE),
    sd_mm = sd(growing_season_precipitation_mm, na.rm = TRUE),
    q25_mm = quantile(growing_season_precipitation_mm, 0.25, na.rm = TRUE),
    q75_mm = quantile(growing_season_precipitation_mm, 0.75, na.rm = TRUE),
    iqr_mm = IQR(growing_season_precipitation_mm, na.rm = TRUE),
    .groups = "drop"
  )

write_csv(
  precipitation_summary,
  here("results", "tables", "precipitation-summary.csv"),
  na = ""
)
~~~

Precipitation is the October-April total of country-area mean daily CHIRPS
estimates, not rainfall measured at maize fields. Do not infer farm exposure
or agronomic thresholds from this summary.

---

## 6. Quantify yield-precipitation association

Create complete pairs and calculate pooled, country-specific, and
period-specific covariance and correlation. A reusable helper keeps the
definition consistent:

~~~r
association_summary <- function(data) {
  paired <- data |>
    filter(
      !is.na(yield_tonnes_per_hectare),
      !is.na(growing_season_precipitation_mm)
    )

  tibble(
    n_pairs = nrow(paired),
    covariance = cov(
      paired$growing_season_precipitation_mm,
      paired$yield_tonnes_per_hectare
    ),
    pearson_correlation = cor(
      paired$growing_season_precipitation_mm,
      paired$yield_tonnes_per_hectare
    )
  )
}

pooled_association <- association_summary(maize) |>
  mutate(scope = "pooled", group = "all", .before = 1)

country_association <- maize |>
  group_by(project_country_id, project_country_name) |>
  group_modify(~ association_summary(.x)) |>
  ungroup() |>
  mutate(scope = "within country", .before = 1)

period_association <- maize |>
  group_by(analysis_period) |>
  group_modify(~ association_summary(.x)) |>
  ungroup() |>
  mutate(scope = "within period", .before = 1)

association <- bind_rows(
  pooled_association,
  country_association,
  period_association
)

write_csv(
  association,
  here("results", "tables", "yield-precipitation-association.csv"),
  na = ""
)
~~~

If either variable has zero variance, correlation is undefined; preserve `NA`
and explain it. Compare pooled results with faceted graphics and
within-country results, and discuss country differences, common trends,
irrigation, temperature, inputs, spatial aggregation, measurement error, and
omitted variables. Do not call correlation an effect.

---

## 7. Assess evidence about stability

Create a compact comparison of recent training and later test periods. Select
the two rows per country, reshape them, and calculate:

~~~r
stationarity_diagnostic <- yield_period_summary |>
  filter(analysis_period != "1990-2005: earlier history") |>
  select(
    project_country_id, project_country_name, analysis_period,
    n, mean_t_per_ha, sd_t_per_ha
  ) |>
  pivot_wider(
    names_from = analysis_period,
    values_from = c(n, mean_t_per_ha, sd_t_per_ha),
    names_sep = "__"
  ) |>
  mutate(
    mean_change_t_per_ha =
      `mean_t_per_ha__2018-2022: later test` -
      `mean_t_per_ha__2006-2017: recent training`,
    sd_ratio =
      `sd_t_per_ha__2018-2022: later test` /
      `sd_t_per_ha__2006-2017: recent training`
  )

write_csv(
  stationarity_diagnostic,
  here("results", "tables", "stationarity-diagnostic.csv"),
  na = ""
)
~~~

The mean change retains yield units. The SD ratio can be unstable with a
small or near-zero denominator, and the test period contains only five
values — do not turn these measures into universal thresholds.

Review them with the full time-series plots, all period summaries,
missingness, source documentation, and period-specific associations. For
each country, describe evidence as "no clear change visible," "possible
level change," "possible variance change," or "insufficient evidence,"
rather than "stationary: yes/no." Formal tests are optional and require a
documented null hypothesis, trend and lag specification, and limitations.

---

## 8. Write the modeling handoff

Create `results/descriptive-modeling-handoff.md` with:

~~~markdown
# Descriptive modeling handoff

## Scope and data
## Coverage
## Yield location, dispersion, and shape
## Period stability
## Precipitation
## Yield-precipitation association
## Implications for explanatory modeling
## Implications for predictive modeling
## Limitations and unresolved questions
~~~

Support statements with named tables or figures. Include a group or period
difference, evidence relevant to stationarity, the difference between pooled
and within-country association, a reason for a time-aware split, structures a
later explanatory model may need, threats to predictive transferability, and
limitations of national FAOSTAT and country-area CHIRPS measures.

Do not select a model yet. State requirements and open questions. Rerun the
script from a clean R session and inspect every artifact:

~~~bash
Rscript scripts/describe-maize-data.R
git status --short
~~~

---

## Independent extension

Choose one extension and document its question, implementation, result, and
limitations.

### Option A: Within-country deviations

Subtract each country's mean from yield and precipitation, then correlate the
deviations. Compare with the pooled correlation and explain which differences
this removes and which temporal confounding remains.

### Option B: Rolling descriptions

For one country, calculate a rolling mean and SD using a documented window.
Explain incomplete initial windows and how window length changes the pattern.

### Option C: Lag-one autocorrelation

For each country, correlate yield at year \(t\) with year \(t-1\). Verify
consecutive years before pairing. Discuss sample size and why autocorrelation
does not identify a mechanism.

### Option D: Period sensitivity

Repeat the comparison with one substantively justified alternative split.
State the justification before calculation and identify robust and sensitive
conclusions.

---

## Troubleshooting

- **A summary returns `NaN`:** the group may contain no non-missing
  observations — count available values and preserve missing results rather
  than replacing them.
- **Correlation returns `NA`:** check complete pairs and whether either
  variable has zero variance.
- **Results differ from a figure:** check input, filters, grouping, period
  labels, units, and missing-value rules; confirm figure and table share the
  same grain.
- **Pooled and country-specific correlations have different signs:** this is
  possible and important — inspect country means, faceted plots, trends, and
  aggregation rather than retaining only the preferred result.
- **The later-period SD changes greatly:** the period contains only five
  observations per country; inspect raw values and report the small
  denominator.
- **A formal test contradicts descriptive evidence:** review its null
  hypothesis, deterministic terms, lag selection, sample size, and
  sensitivity to structural breaks, and report the disagreement.
- **The script overwrites analytical data:** stop. Descriptive analysis
  reads derived data and writes results; it must not modify
  `data/managed/` or `data/derived/`.

---

## Completion checklist

- [ ] Population, sample, grain, groups, periods, variables, and units are stated.
- [ ] Coverage and missingness accompany every analytical scope.
- [ ] Every statistic reports an effective observation or pair count.
- [ ] Mean and median are interpreted with SD, IQR, range, and graphics.
- [ ] Country-specific summaries are compared with pooled results.
- [ ] Period definitions follow the documented workflow.
- [ ] Yield and precipitation preserve their measurement contracts.
- [ ] Covariance and correlation are interpreted as association, not causation.
- [ ] Pooled, within-country, and period-specific associations are compared.
- [ ] Stationarity evidence includes level, variation, time plots, and association drift.
- [ ] No diagnostic is presented as proof of stationarity.
- [ ] The later test period's small sample size is visible.
- [ ] One project-relative script recreates all tables.
- [ ] The handoff distinguishes explanatory and predictive implications.
- [ ] Managed and derived inputs remain unchanged.

---

## Reflect on the application

1. Which measures best represented each country's yield distribution?
2. Which finding was hidden by a pooled summary?
3. Where was there evidence of changing level or variation?
4. What appeared stable, and why is that not proof of stationarity?
5. How did pooled and within-country associations differ?
6. Why is a time-aware test period preferable to a random split?
7. Which finding threatens predictive transferability?

---

## Further resources

- [R for Data Science (2e): Data transformation](https://r4ds.hadley.nz/data-transform.html)
- [`dplyr::summarise()` reference](https://dplyr.tidyverse.org/reference/summarise.html)
- [OpenIntro Statistics](https://www.openintro.org/book/os/)
- [NIST/SEMATECH e-Handbook: Exploratory Data Analysis](https://www.itl.nist.gov/div898/handbook/eda/eda.htm)
- [Forecasting: Principles and Practice — Stationarity and differencing](https://otexts.com/fpp3/stationarity.html)
- [The Turing Way: Data visualisation](https://book.the-turing-way.org/communication/visualisation/)
