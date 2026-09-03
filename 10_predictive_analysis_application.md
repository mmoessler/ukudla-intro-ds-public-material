# Evaluate predictive models for maize yield

---

- Last Update: 2026-08-25
- Source: [10_predictive_analysis_application.md](/learning-modules/intro-ds-module/10_predictive_analysis_application.md)
- Estimated completion time: 6–8 hours
- Independent extension: 2–3 hours
- Prerequisites: Motivation and Concepts pages; completed Descriptive Data Analysis and Explanatory Modeling workflows
- Required output: prediction contract, split audit, reproducible benchmark predictions, overall and country-level evaluation, diagnostic figure, and bounded conclusion

---

## Outline

- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Scenario and deliverables](#scenario-and-deliverables)
- [Before you begin](#before-you-begin)
- [1. Write the prediction contract](#1-write-the-prediction-contract)
- [2. Audit the temporal split](#2-audit-the-temporal-split)
- [3. Inspect the candidate baselines](#3-inspect-the-candidate-baselines)
- [4. Run the predictive workflow](#4-run-the-predictive-workflow)
- [5. Reconstruct and verify the metrics](#5-reconstruct-and-verify-the-metrics)
- [6. Diagnose errors by country and year](#6-diagnose-errors-by-country-and-year)
- [7. Compare evidence with intended use](#7-compare-evidence-with-intended-use)
- [8. Write a bounded predictive conclusion](#8-write-a-bounded-predictive-conclusion)
- [Independent extension](#independent-extension)
- [Troubleshooting](#troubleshooting)
- [Completion checklist](#completion-checklist)
- [Reflect on the application](#reflect-on-the-application)
- [Further resources](#further-resources)

---

## Learning objectives

After completing this exercise, you should be able to:

- document a predictive task before evaluating models;
- verify that training and test observations follow the intended temporal split;
- explain which structure each benchmark model adds;
- confirm that test outcomes are excluded from model fitting;
- generate and validate reproducible row-level predictions;
- independently calculate and compare MAE and RMSE;
- inspect errors overall, by country, and over time;
- recognize the limits of a five-year, known-country test design;
- distinguish predictive evidence from causal evidence; and
- communicate a bounded recommendation for model use.

---

## Place in the session

This is the **Application** part of the Predictive Modeling session:

~~~text
Motivation  →  Concepts  →  Application
                              ↑
                           this page
~~~

Before beginning, review [Why predictive modeling requires unseen-data
evaluation](10_predictive_analysis_motivation.md) and [Understand
predictive-modeling concepts](10_predictive_analysis_concepts.md).

The Explanatory Modeling workflow asked whether modeled relationships could
support a causal interpretation. This exercise preserves a separate script,
objective, and evidence standard. It evaluates later observations and does not
interpret predictive coefficients as causal effects.

---

## Scenario and deliverables

The project team wants a transparent benchmark for predicting recently
observed national maize yields:

> When models are fitted using country-year observations from 1990–2017, how
> accurately do simple procedures predict log maize yield for the same
> countries in 2018–2022?

The project already contains a compact implementation. Audit it as a predictive
workflow, reproduce its results, extend its evaluation, and state what the
evidence supports.

Use or create these artifacts:

~~~text
docs/predictive-modeling.md
scripts/model-maize-yield.R
results/tables/maize-yield-predictions.csv
results/tables/model-performance.csv
results/tables/predictive-performance-by-country.csv
figures/predictive-observed-versus-predicted.png
results/predictive-modeling-conclusion.md
~~~

The first two generated tables belong to the starting workflow. Add the
country-level table, diagnostic figure, and conclusion reproducibly rather
than editing outputs by hand. If the repository implementation later uses
different final paths, document the mapping and preserve the same evidence.

---

## Before you begin

Work from the standalone `maize-yield-project` repository. Confirm the branch
and working tree:

~~~bash
git status --short --branch
~~~

Restore the declared R environment if required:

~~~r
renv::restore()
renv::status()
~~~

The predictive input is `data/derived/maize-yield-panel.csv`. Do not edit it
manually. It should be recreated through the earlier acquisition, integration,
preparation, and validation stages. Review:

~~~text
README.md
docs/data-preparation.md
docs/descriptive-data-analysis.md
docs/explanatory-modeling.md
scripts/functions.R
scripts/model-maize-yield.R
~~~

Before running the script, predict which candidate will perform best and write
one reason why that expectation could fail under temporal change.

---

## 1. Write the prediction contract

Create `docs/predictive-modeling.md`, or add an equivalent section if it
already exists. Record the contract before inspecting test outcomes.

| Component | Core teaching decision |
| --- | --- |
| Target | Annual national `log_yield` |
| Prediction unit | One country-year |
| Target population | Project countries represented during training |
| Training period | 1990–2017 |
| Test period | 2018–2022 |
| Information set | Country identity and calendar year |
| Candidates | Historical mean, common trend, country-plus-time model |
| Evidence | Row-level held-out errors, MAE, and RMSE |
| Intended use | Teaching benchmark for later-period prediction |
| Excluded uses | Operational forecasts, new-country transfer, causal inference |

Add an explicit statement about prediction time. The current task does not
define a real issue date or operational horizon. Describe it as a held-out
later-period benchmark rather than a validated pre-season forecast.

Answer before continuing:

1. Why is precipitation absent from the core information set?
2. What must be defined before adding it?
3. Which loss would matter if large misses were especially costly?
4. Does the test assess later years, new countries, or both?

Do not revise the contract silently after seeing which model wins.

---

## 2. Audit the temporal split

Read the prepared data and inspect its grain and coverage:

~~~r
library(dplyr)
library(readr)

maize <- read_csv(
  "data/derived/maize-yield-panel.csv",
  show_col_types = FALSE
)

maize |>
  summarise(
    rows = n(),
    countries = n_distinct(country),
    first_year = min(year),
    last_year = max(year),
    missing_log_yield = sum(is.na(log_yield))
  )

split_audit <- maize |>
  mutate(partition = if_else(year <= 2017, "training", "test")) |>
  count(partition, country, name = "rows") |>
  arrange(partition, country)

split_audit
~~~

Verify these invariants:

- each row represents one country-year;
- training ends in 2017 and testing starts in 2018;
- every test country also exists in training;
- no country-year appears twice;
- the target is present for evaluation but unused during fitting; and
- no test observation enters a training summary or model call.

~~~r
stopifnot(!anyDuplicated(maize[c("country", "year")]))

training <- maize |> filter(year <= 2017)
testing <- maize |> filter(year >= 2018)

stopifnot(max(training$year) == 2017)
stopifnot(min(testing$year) == 2018)
stopifnot(all(unique(testing$country) %in% unique(training$country)))
~~~

Write the audit to `results/tables/` if another generated artifact does not
already record it. Explain why a random country-year split answers a different
question: it can train on later years while evaluating earlier ones and put
nearby observations from one country on both sides.

The temporal split does not test transfer to an unseen country. The country
model requires country levels already represented during training.

---

## 3. Inspect the candidate baselines

Open `scripts/model-maize-yield.R` and identify every object learned from the
training data. The core implementation corresponds to:

~~~r
historical_mean <- mean(training$log_yield, na.rm = TRUE)
trend_model <- lm(log_yield ~ year, data = training)
country_model <- lm(log_yield ~ year + country, data = training)
~~~

| Candidate | Learned information | Main limitation |
| --- | --- | --- |
| Historical mean | One average log yield | Ignores country and time |
| Common trend | Intercept and common year slope | Ignores country differences |
| Country plus time | Country differences and common slope | Assumes common linear time change |

Confirm that each object is fitted only from `training`. Testing may supply
country and year to `predict()`, but `testing$log_yield` must not enter fitting
or preprocessing.

Write down your expected ranking. Country differences may favor the country
model, but different slopes or later-period shifts may make extrapolation fail.
A few large errors can also affect RMSE more strongly than MAE.

---

## 4. Run the predictive workflow

From the project root, run:

~~~bash
Rscript scripts/model-maize-yield.R
~~~

Alternatively, run the complete pipeline when source data and dependencies are
available:

~~~bash
Rscript scripts/run-all.R
~~~

Inspect outputs rather than trusting a successful exit code:

~~~bash
ls -lh results/tables/maize-yield-predictions.csv \
  results/tables/model-performance.csv
~~~

Verify completeness in R:

~~~r
predictions <- read_csv(
  "results/tables/maize-yield-predictions.csv",
  show_col_types = FALSE
)
performance <- read_csv(
  "results/tables/model-performance.csv",
  show_col_types = FALSE
)

stopifnot(nrow(predictions) == nrow(testing))
stopifnot(!anyDuplicated(predictions[c("country", "year")]))
stopifnot(all(predictions$year >= 2018))

performance
~~~

Check required target and prediction columns explicitly for missing values.
Record the lowest-MAE and lowest-RMSE candidates, whether both metrics agree,
and the improvement over the historical mean. Do not call the winner
“accurate” without a decision-relevant acceptable-error threshold.

---

## 5. Reconstruct and verify the metrics

Recalculate metrics independently from row-level predictions:

~~~r
candidate_columns <- c(
  "historical_mean",
  "linear_trend",
  "country_model"
)

verified_performance <- lapply(candidate_columns, function(candidate) {
  errors <- predictions$log_yield - predictions[[candidate]]

  tibble(
    model = candidate,
    observations = sum(!is.na(errors)),
    mean_error = mean(errors, na.rm = TRUE),
    mae = mean(abs(errors), na.rm = TRUE),
    rmse = sqrt(mean(errors^2, na.rm = TRUE))
  )
}) |>
  bind_rows()

verified_performance
~~~

Compare these values with `model-performance.csv` using a floating-point
tolerance rather than text equality. Interpret MAE as the average absolute miss
and RMSE as a measure giving large misses more influence. Both remain on the
log-yield scale. Mean error diagnoses systematic underprediction when positive
and overprediction when negative under this convention.

If MAE and RMSE rank models differently, locate the largest errors. Their
disagreement reflects different loss functions and is not automatically a
software failure.

---

## 6. Diagnose errors by country and year

Reshape predictions and calculate country-level diagnostics:

~~~r
library(tidyr)

prediction_long <- predictions |>
  pivot_longer(
    cols = all_of(candidate_columns),
    names_to = "model",
    values_to = "estimate"
  ) |>
  mutate(
    error = log_yield - estimate,
    absolute_error = abs(error),
    squared_error = error^2
  )

performance_by_country <- prediction_long |>
  group_by(model, country) |>
  summarise(
    observations = n(),
    mean_error = mean(error),
    mae = mean(absolute_error),
    rmse = sqrt(mean(squared_error)),
    .groups = "drop"
  )

write_csv(
  performance_by_country,
  "results/tables/predictive-performance-by-country.csv"
)
~~~

Every country has only five test observations, so treat patterns as diagnostic.
Ask whether the overall winner fails for one country, whether errors have a
consistent direction, and whether one country-year drives RMSE.

Create an observed-versus-predicted plot:

~~~r
library(ggplot2)

prediction_plot <- ggplot(
  prediction_long,
  aes(x = year, group = model, colour = model)
) +
  geom_line(aes(y = estimate), linewidth = 0.7) +
  geom_point(aes(y = estimate), size = 1.5) +
  geom_line(aes(y = log_yield, group = 1), colour = "black") +
  geom_point(aes(y = log_yield), colour = "black") +
  facet_wrap(~ country, scales = "free_y") +
  labs(
    title = "Observed and predicted maize yield in the test period",
    subtitle = "Black: observed log yield; colour: benchmark prediction",
    x = "Year", y = "Log yield", colour = "Model"
  ) +
  theme_minimal()

ggsave(
  "figures/predictive-observed-versus-predicted.png",
  prediction_plot,
  width = 10, height = 7, dpi = 300
)
~~~

Adapt aesthetics to project conventions while preserving target scale, test
period, and model identity in the labels.

---

## 7. Compare evidence with intended use

Return to the contract and assess alignment:

- **Population:** the test contains later observations from known countries,
  not unseen countries or farms.
- **Time:** one five-year period provides limited evidence about later
  extrapolation and none about unprecedented future conditions.
- **Information:** country and year are available, but are proxies rather than
  measurements of agricultural mechanisms.
- **Loss:** MAE and RMSE quantify log-scale errors, but no policy-specific cost
  or tolerance has been supplied.
- **Causality:** useful prediction does not show that country, year, or an added
  precipitation variable causes yield.

The comparison can rank transparent benchmarks for this test. It cannot by
itself certify an operational crop forecast or a high-stakes decision system.

---

## 8. Write a bounded predictive conclusion

Generate `results/predictive-modeling-conclusion.md` from the analysis script
or report step. Include:

1. target, unit, training period, and test period;
2. candidates and information set;
3. held-out observations and countries;
4. MAE and RMSE for every candidate;
5. improvement over the historical-mean baseline;
6. important country- or year-specific errors;
7. whether ranking depends on the metric;
8. intended and excluded uses; and
9. conditions requiring reevaluation.

Use this pattern:

> Among the predefined procedures, **[candidate]** produced the lowest
> **[metric]** for **[n]** held-out country-years from 2018–2022, improving on
> the historical-mean baseline by **[amount]**. Performance differed by
> **[country/year pattern]**, and the short test period provides limited
> evidence. The result supports this model only as a reproducible benchmark
> for later observations from the represented countries. It does not validate
> an operational forecast, transfer to new countries, or a causal claim.

Replace every placeholder with generated evidence. If complexity does not beat
the baseline, report that result directly.

---

## Independent extension

Choose **one** extension and keep the test set protected.

### Option A: Add a country-mean baseline

Calculate country means from training only, join them to test rows, and compare
errors. Explain how this differs from the country-plus-time model.

### Option B: Add rolling-origin validation

Within 1990–2017, define expanding training windows and evaluate subsequent
years. Examine whether rankings are stable while leaving 2018–2022 untouched
for the final assessment.

### Option C: Define a precipitation-based task

Specify whether the task is pre-season forecasting, in-season updating, or
post-season estimation. Identify which CHIRPS observations exist at the issue
date, create features without leakage, and compare the added candidate.

### Option D: Evaluate original-scale yield

Define whether the desired quantity is a conditional median or mean after
back-transformation. Justify the transformation and calculate original-scale
metrics without mixing scales in one ranking.

For every extension, update the contract, preserve row-level predictions,
compare with the same relevant baseline, and document whether conclusions
change.

---

## Troubleshooting

### `renv` reports that the project is out of sync

Run `renv::status()` and restore from `renv.lock` when packages are missing.
Do not update the lockfile merely to suppress the message.

### The prepared panel is missing

Run preceding acquisition, validation, integration, and preparation stages or
the complete pipeline. Do not copy a generated panel without documenting its
provenance.

### A test country is new to the country model

The core design expects every test country in training. Do not silently coerce
the factor or drop the country. Decide whether the task now concerns
new-country transfer and adopt an appropriate grouped evaluation.

### Predictions contain missing values

Check missing predictors, factor levels, formulas, and `predict()` output.
Validate each candidate column before metrics and report exclusions rather than
silently changing denominators.

### Reconstructed metrics differ

Confirm the target, prediction columns, error convention, missing-value rows,
log scale, test rows, and RMSE formula.

### The best model differs by metric or country

Inspect large residuals and relate metric choice to intended loss. Report the
instability rather than hiding it behind one summary.

### Quarto cannot find a report

Run from the repository root and check report paths. In a container, ensure the
repository—including changed `scripts/` and `reports/`—is present or mounted at
the expected working directory. Mounting only data and output directories does
not update scripts baked into an older image.

---

## Completion checklist

- [ ] The contract fixes target, unit, information, split, metrics, and use.
- [ ] Training ends in 2017 and testing begins in 2018.
- [ ] Every test country occurs in training.
- [ ] Models and learned operations use training data only.
- [ ] The test set was not used to invent or tune candidates.
- [ ] Row-level predictions are keyed by country and year.
- [ ] MAE and RMSE were independently verified.
- [ ] Performance was inspected overall and by country or year.
- [ ] A diagnostic figure shows observed and predicted outcomes.
- [ ] Sample sizes and target scale are reported.
- [ ] The conclusion compares candidates with a baseline.
- [ ] Unsupported operational and causal claims are excluded.
- [ ] Artifacts can be recreated from documented code and environment files.
- [ ] Repository changes were reviewed before committing.

---

## Reflect on the application

1. Which workflow decision most protects the credibility of performance?
2. How would the task change for a country absent from training?
3. Is the best candidate's gain stable across countries?
4. Which metric reflects the intended error cost, and what is missing to decide?
5. Does descriptive evidence about shifts explain predictive failures?
6. Which operations would leak information if learned from the complete panel?
7. What issue date makes CHIRPS precipitation a legitimate predictor?
8. What evidence is needed before operational use?
9. How does the predictive conclusion differ from the causal conclusion?
10. Which artifacts let another analyst audit the result?

---

## Further resources

- [Tidy Modeling with R](https://www.tmwr.org/) provides practical guidance for reproducible splitting, resampling, recipes, fitting, and evaluation in R.
- [Forecasting: Principles and Practice](https://otexts.com/fpp3/) explains time-aware evaluation, residual checks, and forecasting limitations.
- [An Introduction to Statistical Learning](https://www.statlearning.com/) introduces statistical learning, regression, resampling, and the bias–variance trade-off.
- [The Turing Way: Reproducible Research](https://book.the-turing-way.org/reproducible-research/reproducible-research/) provides guidance for auditable computational workflows and documentation.
- [FAO crop and livestock statistics](https://www.fao.org/statistics/data-dissemination/crop-and-livestock-statistics/en) supplies context for the source yield data.
- [CHIRPS](https://www.chc.ucsb.edu/data/chirps) documents precipitation data available for a carefully defined extension.

Keep the conclusion narrow: this exercise evaluates transparent prediction
procedures on one later period. Its value lies in honest, reproducible
evaluation—not in claiming an operational forecasting system.
