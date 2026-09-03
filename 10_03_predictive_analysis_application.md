# 10.3) Apply predictive analysis: maize-yield worked example

---

- Source: [10_03_predictive_analysis_application.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/10_03_predictive_analysis_application.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/10_03_predictive_analysis_application.md)
- Feedback: [Topic 10: Predictive Analysis](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/11)
- Estimated completion time: 6–8 hours; independent extension: 2–3 hours
- Prerequisites: Motivation and Concepts pages; Descriptive Data Analysis and Explanatory Modeling workflows
- Required output: prediction contract, split audit, benchmark predictions, evaluation, diagnostic figure, and bounded conclusion

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Scenario and deliverables](#scenario-and-deliverables)
  - [Transfer the workflow to another study design](#transfer-the-workflow-to-another-study-design)
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
- generate and validate reproducible row-level predictions;
- independently calculate, compare, and diagnose MAE and RMSE by country and year;
- recognize the limits of a five-year, known-country test design; and
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
evaluation](10_01_predictive_analysis_motivation.md) and [Understand
predictive-modeling concepts](10_02_predictive_analysis_concepts.md).

The Explanatory Modeling workflow asked whether modeled relationships could
support a causal interpretation. This exercise preserves a separate script,
objective, and evidence standard: it evaluates later observations and does
not interpret predictive coefficients as causal effects.

---

## Scenario and deliverables

> **Worked-example scope:** The supplied task predicts later country-year
> observations. Other projects may predict new samples, plots, sites, farms,
> batches, or future measurement occasions. The evaluation split must reproduce
> that intended use rather than copy the example's calendar-year split.

The project team wants a transparent benchmark for predicting recently
observed national maize yields:

> When models are fitted using country-year observations from 1990–2017, how
> accurately do simple procedures predict log maize yield for the same
> countries in 2018–2022?

The project already contains a compact implementation. Audit it as a
predictive workflow, reproduce and extend its evaluation, and state what the
evidence supports.

Use or create these artifacts:

~~~text
docs/predictive-modeling.md
scripts/predict-maize-yield.R
results/tables/maize-yield-predictions.csv
results/tables/model-performance.csv
results/tables/predictive-performance-by-country.csv
figures/predictive-observed-versus-predicted.png
results/predictive-modeling-conclusion.md
~~~

### Transfer the workflow to another study design

| Intended use | Suitable evaluation boundary | Leakage to prevent |
| --- | --- | --- |
| New laboratory samples | Hold out independent biological samples or future batches | Aliquots or technical replicates of one sample crossing splits; batch-wide preprocessing |
| New experimental units | Hold out units, blocks, sites, or a later experiment as use requires | Repeated measures or treatment summaries shared across splits |
| New observational units or visits | Split by unit, site, cluster, or time to match deployment | Records from one unit in both sets; post-outcome or future information |
| New secondary-data periods or entities | Hold out later periods, entities, or both | Revisions, future aggregates, or preprocessing learned from the test domain |

Define the target, prediction time, information availability, deployment
population, and cost of errors before choosing the split or metric. A random
row split is appropriate only when future use truly concerns exchangeable,
independent rows from the same population.

The first two generated tables belong to the starting workflow; add the
country-level table, diagnostic figure, and conclusion reproducibly rather
than editing outputs by hand.

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
manually — recreate it through the earlier acquisition, integration,
preparation, and validation stages. Review:

~~~text
README.md
docs/data-preparation.md
docs/descriptive-data-analysis.md
docs/explanatory-modeling.md
scripts/functions.R
scripts/predict-maize-yield.R
~~~

Before running the script, predict which candidate will perform best and
write one reason that expectation could fail under temporal change.

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

Add an explicit statement about prediction time: the task does not define a
real issue date or operational horizon, so describe it as a held-out
later-period benchmark rather than a validated pre-season forecast.

Answer before continuing: why is precipitation absent from the core
information set, what must be defined before adding it, and does the test
assess later years, new countries, or both? Do not revise the contract
silently after seeing which model wins.

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

Verify these invariants: each row represents one country-year; training ends
in 2017 and testing starts in 2018; every test country also exists in
training; no country-year appears twice; and no test observation enters a
training summary or model call.

~~~r
stopifnot(!anyDuplicated(maize[c("country", "year")]))

training <- maize |> filter(year <= 2017)
testing <- maize |> filter(year >= 2018)

stopifnot(max(training$year) == 2017)
stopifnot(min(testing$year) == 2018)
stopifnot(all(unique(testing$country) %in% unique(training$country)))
~~~

Write the audit to `results/tables/` if not already recorded. A random
country-year split would answer a different question — training on later
years while evaluating earlier ones, with nearby observations from one
country on both sides. The temporal split also does not test transfer to an
unseen country, since the country model requires levels already seen in
training.

---

## 3. Inspect the candidate baselines

Open `scripts/predict-maize-yield.R` and identify every object learned from
training data:

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
country and year to `predict()`, but `testing$log_yield` must not enter
fitting or preprocessing.

Write down your expected ranking: country differences may favor the country
model, but different slopes or later-period shifts may make extrapolation
fail, and a few large errors can affect RMSE more strongly than MAE.

---

## 4. Run the predictive workflow

From the project root, run:

~~~bash
Rscript scripts/predict-maize-yield.R
~~~

Alternatively, run the complete pipeline when dependencies are available:

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

Check required columns for missing values, and record the lowest-MAE and
lowest-RMSE candidates and the improvement over the historical mean — but do
not call the winner “accurate” without a decision-relevant threshold.

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
tolerance. MAE is the average absolute miss; RMSE weights large misses more;
both remain on the log-yield scale, and mean error diagnoses systematic
under- or overprediction.

If MAE and RMSE rank models differently, locate the largest errors — their
disagreement reflects different loss functions, not automatically a software
failure.

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

Every country has only five test observations, so treat patterns as
diagnostic: ask whether the overall winner fails for one country, whether
errors have a consistent direction, and whether one country-year drives
RMSE.

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

Adapt aesthetics to project conventions while preserving scale, period, and
model identity in the labels.

---

## 7. Compare evidence with intended use

Return to the contract and assess alignment:

- **Population:** later observations from known countries, not unseen
  countries or farms.
- **Time:** one five-year period gives limited evidence about later
  extrapolation and none about unprecedented conditions.
- **Information:** country and year are proxies, not measurements of
  agricultural mechanisms.
- **Loss:** MAE and RMSE quantify log-scale errors without a policy-specific
  cost or tolerance.
- **Causality:** useful prediction does not show that country, year, or
  precipitation causes yield.

The comparison can rank transparent benchmarks for this test; it cannot by
itself certify an operational crop forecast or a high-stakes decision system.

---

## 8. Write a bounded predictive conclusion

Generate `results/predictive-modeling-conclusion.md` from the analysis
script or report step. Include the contract's key fields, MAE and RMSE for
every candidate, the improvement over the historical-mean baseline, notable
country- or year-specific errors, whether ranking depends on the metric, and
conditions requiring reevaluation.

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

Choose **one** extension, keep the test set protected, update the contract,
and compare against the same baseline:

- **Country-mean baseline:** calculate country means from training only,
  join them to test rows, and compare with the country-plus-time model.
- **Rolling-origin validation:** within 1990–2017, define expanding training
  windows and check whether rankings stay stable, leaving 2018–2022
  untouched for the final assessment.
- **Precipitation-based task:** specify pre-season, in-season, or
  post-season framing, identify which CHIRPS observations exist at the issue
  date, and add the feature without leakage.
- **Original-scale yield:** define and justify a back-transformation to a
  conditional median or mean, and evaluate it without mixing scales in one
  ranking.

---

## Troubleshooting

- **`renv` out of sync:** run `renv::status()` and restore from `renv.lock`
  rather than editing the lockfile to suppress the message.
- **Prepared panel missing:** run the preceding acquisition, validation,
  integration, and preparation stages rather than copying an undocumented
  panel.
- **Test country new to the country model:** decide whether the task now
  concerns new-country transfer and adopt a grouped evaluation, rather than
  silently coercing or dropping it.
- **Predictions contain missing values:** check predictors, factor levels,
  formulas, and `predict()` output, and report exclusions explicitly.
- **Reconstructed metrics or rankings disagree:** confirm the target, error
  convention, log scale, test rows, and RMSE formula, then inspect large
  residuals instead of one summary.
- **Quarto cannot find a report:** run from the repository root; in a
  container, ensure changed `scripts/` and `reports/` are mounted, not
  baked into an older image.

---

## Completion checklist

- [ ] The contract fixes target, unit, information, split, metrics, and use.
- [ ] Training ends in 2017 and testing begins in 2018, and every test
      country occurs in training.
- [ ] Models and learned operations use training data only, and the test
      set was not used to invent or tune candidates.
- [ ] Row-level predictions are keyed by country and year.
- [ ] MAE and RMSE were independently verified.
- [ ] Performance was inspected overall and by country or year, with a
      diagnostic figure and reported sample sizes and target scale.
- [ ] The conclusion compares candidates with a baseline and excludes
      unsupported operational and causal claims.
- [ ] Artifacts can be recreated from documented code and environment
      files, and repository changes were reviewed before committing.

---

## Reflect on the application

1. Which workflow decision most protects the credibility of performance?
2. How would the task change for a country absent from training?
3. Is the best candidate's gain stable across countries?
4. Which metric reflects the intended error cost, and what is missing to decide?
5. Which operations would leak information if learned from the complete panel?
6. What evidence is needed before operational use, and how does that differ
   from the causal conclusion?
7. Which artifacts let another analyst audit the result?

---

## Further resources

- [Tidy Modeling with R](https://www.tmwr.org/) covers reproducible splitting, fitting, and evaluation in R.
- [Forecasting: Principles and Practice](https://otexts.com/fpp3/) explains time-aware evaluation and forecasting limitations.
- [An Introduction to Statistical Learning](https://www.statlearning.com/) introduces regression, resampling, and the bias–variance trade-off.
- [FAO crop and livestock statistics](https://www.fao.org/statistics/data-dissemination/crop-and-livestock-statistics/en) supplies context for the source yield data.
- [CHIRPS](https://www.chc.ucsb.edu/data/chirps) documents precipitation data available for a carefully defined extension.

Keep the conclusion narrow: this exercise evaluates transparent prediction
procedures on one later period, and its value lies in honest, reproducible
evaluation — not in claiming an operational forecasting system.
