# 9.3) Apply explanatory analysis: maize-yield worked example

---

- Last Update: 2026-09-03
- Source: [09_03_explanatory_analysis_application.md](/learning-modules/intro-ds-module/09_03_explanatory_analysis_application.md)
- Estimated completion time: 6–8 hours
- Independent extension: 2–3 hours
- Prerequisites: Motivation and Concepts pages; completed Descriptive Data Analysis workflow
- Required output: causal question, DAG, identification assessment, modeling script, model/diagnostic tables, bounded conclusion

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Scenario and deliverables](#scenario-and-deliverables)
  - [Transfer the workflow to another study design](#transfer-the-workflow-to-another-study-design)
- [Before you begin](#before-you-begin)
- [1. Define the causal question and estimand](#1-define-the-causal-question-and-estimand)
- [2. Draw the causal diagram](#2-draw-the-causal-diagram)
- [3. Assess identification before estimation](#3-assess-identification-before-estimation)
- [4. Inspect exposure support](#4-inspect-exposure-support)
- [5. Fit the planned regression sequence](#5-fit-the-planned-regression-sequence)
- [6. Interpret estimates and uncertainty](#6-interpret-estimates-and-uncertainty)
- [7. Diagnose models and test specification sensitivity](#7-diagnose-models-and-test-specification-sensitivity)
- [8. Write a bounded causal conclusion](#8-write-a-bounded-causal-conclusion)
- [Independent extension](#independent-extension)
- [Troubleshooting](#troubleshooting)
- [Completion checklist](#completion-checklist)
- [Reflect on the application](#reflect-on-the-application)
- [Further resources](#further-resources)

---

## Learning objectives

After completing this exercise, you should be able to:

- define a target population, exposure contrast, outcome, time horizon, and causal estimand;
- encode causal assumptions in a directed acyclic graph and assess identification requirements;
- fit and compare transparent linear-regression specifications in R;
- inspect residual patterns, influential observations, temporal structure, and exposure support;
- separate model uncertainty from causal-identification uncertainty; and
- conclude whether the results support a causal effect or only adjusted associations.

---

## Place in the session

This is the **Application** part of the Explanatory Modeling session:

~~~text
Motivation  →  Concepts  →  Application
                              ↑
                           this page
~~~

Before beginning, review [Why explanatory modeling requires causal
reasoning](09_01_explanatory_analysis_motivation.md) and [Understand
explanatory-modeling concepts](09_02_explanatory_analysis_concepts.md).

Descriptive Data Analysis established coverage, summaries, and association
evidence. This exercise uses that evidence to design and challenge a causal
analysis — it does not overwrite data or treat prediction and causal
explanation as the same task.

---

## Scenario and deliverables

> **Worked-example scope:** The precipitation question is intentionally used to
> expose the limits of causal inference from aggregated observational data. It
> is not the default causal design. A randomized field or laboratory experiment
> may support stronger identification, while an observational field study
> requires its own defensible comparison and adjustment strategy.

The project team asks:

> What is the causal effect of growing-season precipitation on national maize
> yield in the selected Southern African countries?

Turn this broad question into an auditable analysis and decide whether the
available data identify the intended effect.

Create:

~~~text
docs/causal-model.md
scripts/explain-maize-yield.R
results/tables/explanatory-exposure-support.csv
results/tables/explanatory-model-estimates.csv
results/tables/explanatory-model-diagnostics.csv
results/tables/explanatory-residual-dependence.csv
results/explanatory-modeling-conclusion.md
reports/explanatory-modeling.qmd
~~~

### Transfer the workflow to another study design

| Project context | Source of the causal contrast | Main identification questions |
| --- | --- | --- |
| Laboratory experiment | Randomized manipulation of samples or experimental units | Was assignment implemented, were outcomes measured comparably, and are spillover and attrition negligible? |
| Field experiment | Randomized treatment within blocks, sites, or clusters | Does analysis honor assignment, blocking, clustering, noncompliance, interference, and missing outcomes? |
| Field observation | Naturally varying exposure or practice | Which pre-exposure causes affect both exposure and outcome, how are they measured, and is overlap adequate? |
| Secondary data | Policy, environmental, or reported exposure | Is the intervention well defined at the recorded grain, and do aggregation, selection, timing, and unmeasured confounding prevent identification? |

For every design, define the intervention, comparator, target population,
outcome, and time horizon before selecting an estimator. Randomization can
support exchangeability; it does not by itself fix measurement error,
noncompliance, attrition, interference, or an ambiguous treatment definition.

The causal-model document records the question, estimand, diagram, variable
roles, and identification judgment. The conclusion must distinguish what the
models estimate from what can be interpreted causally.

---

## Before you begin

Work from the standalone `maize-yield-project` repository:

~~~bash
pwd
git status --short --branch
~~~

Restore the environment and recreate upstream evidence:

~~~bash
Rscript scripts/setup.R
Rscript scripts/validate-data.R
Rscript scripts/prepare-maize-data.R
Rscript scripts/integrate-data.R
Rscript scripts/visualize-maize-data.R
Rscript scripts/describe-maize-data.R
~~~

Review:

| Evidence | File |
| --- | --- |
| Integrated-data definition | `docs/data/maize-yield-with-precipitation.md` |
| Variable dictionary | `metadata/maize-yield-with-precipitation-data-dictionary.csv` |
| Visual relationship | `figures/yield-versus-precipitation.png` |
| Country/period descriptions | `results/tables/maize-yield-period-summary.csv` |
| Association by scope | `results/tables/yield-precipitation-association.csv` |
| Stability evidence | `results/tables/stationarity-diagnostic.csv` |
| Modeling handoff | `results/descriptive-modeling-handoff.md` |

The input contains 297 country-years, not randomized rainfall treatments or
field-level exposures.

---

## 1. Define the causal question and estimand

Create `docs/causal-model.md` and complete:

| Element | Provisional definition |
| --- | --- |
| Target population | Country-years for nine selected countries, 1990–2022 |
| Unit | One country-year |
| Exposure | October-April country-area CHIRPS precipitation total |
| Contrast | 100 mm higher versus the observed reference level |
| Outcome | National maize yield (t/ha) for the ending year |
| Time zero / follow-up | Start of growing season / ending-year yield |
| Estimand | Average difference in potential yield under the two exposures |

Then challenge every row: explain why "100 mm higher" does not specify
rainfall timing, intensity, location, or mechanism, and decide whether the
contrast is a provisional scientific target rather than a well-defined
intervention.

Write the potential-outcomes expression:

\[
E\left[Y(P+100)-Y(P)\right]
\]

State whether a constant shift is meaningful throughout the observed range.
Do not change the estimand after inspecting which model is significant.

---

## 2. Draw the causal diagram

Begin with concepts, not only available columns. Include at least:
precipitation amount, timing, and intensity; seasonal weather; national maize
yield; irrigation, soils, and maize-growing area; varieties, fertilizer,
labor, and management; pests and disease; policy, markets, and reporting
practices; country context and calendar time; and measurement and selection
processes.

For every arrow, write one sentence explaining the assumed mechanism and
temporal order, and mark whether each node is measured, proxied, or
unmeasured. Use a simplified diagram in the report while retaining the full
inventory in `docs/causal-model.md`:

~~~text
seasonal weather ───► precipitation ───► maize yield
       │                                      ▲
       └──────────────────────────────────────┘

country/time context ─► precipitation
          │                    │
          └───────────────────►yield

irrigation, soils, inputs and management ────►yield
~~~

This is not a finished adjustment strategy — discuss omitted factors rather
than using the diagram merely to justify variables already in the table.

---

## 3. Assess identification before estimation

Add an identification table:

| Requirement | Evidence | Judgment | Consequence |
| --- | --- | --- | --- |
| Consistency | Seasonal total; timing/spatial versions hidden | Doubtful | 100 mm contrast is ambiguous |
| Exchangeability | Country/year observed; weather and management causes omitted | Not established | Coefficient may remain confounded |
| Positivity | 33 obs. per country; ranges differ | Must inspect | Some contrasts require extrapolation |
| No interference | Countries linked by markets, water, shocks | Uncertain | Treatment representation may be incomplete |
| Measurement | National yield, country-area rainfall | Limited | Exposure-outcome alignment imperfect |
| Selection | Nine countries, complete teaching data | Limited target | Do not generalize automatically |

Do not write "assumption met" merely because it cannot be tested — state what
evidence would be needed, and decide before modeling whether the dataset
plausibly identifies the estimand. The core expectation is that it does not
fully do so; the statistical models remain useful for quantifying how
associations change under selected adjustments and for demonstrating the
consequences of the design limitations.

---

## 4. Inspect exposure support

Create `scripts/explain-maize-yield.R`:

~~~r
# Conduct the project explanatory-modeling analysis.

source("scripts/functions.R")
assert_project_root()
ensure_project_directories()
check_required_packages(c("dplyr", "here", "readr", "tibble"))

library(dplyr)
library(here)
library(readr)
library(tibble)

input_file <- here(
  "data", "derived", "maize-yield-with-precipitation.csv"
)
maize <- read_csv(input_file, show_col_types = FALSE)

required <- c(
  "project_country_id", "project_country_name", "year",
  "yield_tonnes_per_hectare", "growing_season_precipitation_mm"
)
if (length(setdiff(required, names(maize))) > 0 ||
    nrow(maize) != 297L ||
    anyDuplicated(maize[c("project_country_id", "year")])) {
  stop("Expected 297 unique country-year observations and required columns.")
}

maize <- maize |>
  mutate(
    country = factor(project_country_name),
    year_centered = year - 1990,
    precipitation_100mm = growing_season_precipitation_mm / 100
  )
~~~

Summarize support:

~~~r
exposure_support <- maize |>
  group_by(project_country_id, project_country_name) |>
  summarise(
    n = sum(!is.na(precipitation_100mm)),
    minimum_100mm = min(precipitation_100mm, na.rm = TRUE),
    q25_100mm = quantile(precipitation_100mm, 0.25, na.rm = TRUE),
    median_100mm = median(precipitation_100mm, na.rm = TRUE),
    q75_100mm = quantile(precipitation_100mm, 0.75, na.rm = TRUE),
    maximum_100mm = max(precipitation_100mm, na.rm = TRUE),
    .groups = "drop"
  )

write_csv(
  exposure_support,
  here("results", "tables", "explanatory-exposure-support.csv"),
  na = ""
)
~~~

Compare country ranges and distributions — a model with country indicators
primarily uses within-country variation. Identify where a 100 mm contrast
implies extrapolation beyond observed support. Range overlap is a necessary
descriptive check, not proof of conditional positivity.

---

## 5. Fit the planned regression sequence

Fit specifications selected before viewing results:

~~~r
models <- list(
  unadjusted = lm(
    yield_tonnes_per_hectare ~ precipitation_100mm,
    data = maize
  ),
  country = lm(
    yield_tonnes_per_hectare ~ precipitation_100mm + country,
    data = maize
  ),
  time = lm(
    yield_tonnes_per_hectare ~ precipitation_100mm + year_centered,
    data = maize
  ),
  country_time = lm(
    yield_tonnes_per_hectare ~
      precipitation_100mm + country + year_centered,
    data = maize
  ),
  nonlinear_sensitivity = lm(
    yield_tonnes_per_hectare ~
      precipitation_100mm + I(precipitation_100mm^2) +
      country + year_centered,
    data = maize
  )
)
~~~

Document the role of every specification:

| Model | Comparison | Purpose | Remaining concern |
| --- | --- | --- | --- |
| Unadjusted | All country-years | Pooled linear association | Country/time differences |
| Country | Within-country slope | Stable country levels | Time-varying confounding |
| Time | Common trend | Common linear change | Country differences |
| Country + time | Within-country trend | Main adjusted association | Omitted causes, dependence |
| Nonlinear | Curved association | Functional-form sensitivity | Same identification limits |

Country and time terms are not a declared sufficient adjustment set — they
are transparent, available proxies used to show how interpretation changes.

---

## 6. Interpret estimates and uncertainty

Extract the linear precipitation term from the first four models:

~~~r
extract_precipitation <- function(model, model_name) {
  coefficient_table <- summary(model)$coefficients
  interval <- confint(model, "precipitation_100mm", level = 0.95)

  tibble(
    model = model_name,
    n = nobs(model),
    estimate_t_per_ha_per_100mm =
      coefficient_table["precipitation_100mm", "Estimate"],
    standard_error =
      coefficient_table["precipitation_100mm", "Std. Error"],
    confidence_low = interval[1],
    confidence_high = interval[2],
    p_value = coefficient_table["precipitation_100mm", "Pr(>|t|)"],
    r_squared = summary(model)$r.squared,
    adjusted_r_squared = summary(model)$adj.r.squared
  )
}

model_estimates <- bind_rows(
  lapply(
    names(models)[1:4],
    function(name) extract_precipitation(models[[name]], name)
  )
)

write_csv(
  model_estimates,
  here("results", "tables", "explanatory-model-estimates.csv"),
  na = ""
)
~~~

Interpret each estimate in tonnes per hectare per 100 mm and compare
direction and magnitude across models before discussing p-values.

The intervals use default `lm` assumptions, including an error structure that
does not represent within-country temporal dependence. They quantify
model-based sampling uncertainty only — not uncertainty from confounding,
measurement, intervention ambiguity, or specification choice. Do not
interpret the linear term from the quadratic model alone: the modeled
difference varies with baseline precipitation, so inspect predicted contrasts
across supported values instead.

---

## 7. Diagnose models and test specification sensitivity

For every model, calculate transparent checks:

~~~r
model_diagnostics <- bind_rows(lapply(names(models), function(name) {
  model <- models[[name]]
  cooks <- cooks.distance(model)
  residual <- residuals(model)

  tibble(
    model = name,
    n = nobs(model),
    residual_sd = sigma(model),
    maximum_absolute_residual = max(abs(residual)),
    maximum_cooks_distance = max(cooks),
    observations_above_4_over_n = sum(cooks > 4 / nobs(model))
  )
}))

write_csv(
  model_diagnostics,
  here("results", "tables", "explanatory-model-diagnostics.csv"),
  na = ""
)
~~~

Inspect, do not merely calculate: residuals versus fitted values, the
residual distribution and scale-location pattern, high-leverage/Cook's-distance
observations, residuals over time within countries, and lag-one residual
correlation by country.

Create `explanatory-residual-dependence.csv` with lag-one pair counts and
correlations for the main `country_time` model, verifying consecutive years
before forming pairs.

Ask whether adjustment substantially changes the precipitation coefficient,
whether a linear form misses a visible curve, whether conclusions are driven
by a small number of country-years, and which findings are stable across
specifications. Diagnostics can reveal statistical misspecification; they
cannot establish exchangeability or correct measurement.

---

## 8. Write a bounded causal conclusion

Create `results/explanatory-modeling-conclusion.md`:

~~~markdown
# Explanatory-modeling conclusion

## Causal question and estimand
## Data and measurement
## Causal diagram and adjustment reasoning
## Identification assessment
## Statistical models and estimates
## Diagnostics and specification sensitivity
## Supported interpretation
## Unsupported interpretations
## Evidence needed for stronger causal inference
~~~

Your conclusion must include:

- the estimated association per 100 mm from every planned model, and how it changes with country and time terms;
- the nonlinear sensitivity result and relevant overlap, influence, residual, and temporal warnings;
- which identification assumptions are doubtful or unassessable, and why default confidence intervals exclude identification uncertainty;
- an explicit statement that correlation and adjusted regression are not automatically causal; and
- concrete data or design improvements.

A defensible conclusion may be:

> The models quantify associations between country-area growing-season
> precipitation and national maize yield conditional on selected country and
> time terms. Because the exposure is not a well-defined intervention and
> important time-varying common causes and measurement limitations remain, the
> available data do not identify the proposed causal effect.

Render `reports/explanatory-modeling.qmd` so the question, diagram,
identification table, estimates, and conclusion stay linked to the
reproducible artifacts.

---

## Independent extension

Choose one extension and document its causal purpose and limitations.

- **Alternative exposure representation:** compare the seasonal total with dry-day counts or sub-season periods; explain the effect on consistency.
- **Country-specific associations:** add precipitation-by-country interactions and explain the small sample size and increased uncertainty.
- **Negative-control reasoning:** propose an exposure or outcome that should share bias mechanisms without a causal link, and explain what a detected association would reveal.
- **Stronger research design:** sketch a field experiment, irrigation intervention, or subnational panel, stating its assignment mechanism, estimand, and practical limits.

---

## Troubleshooting

- **Coefficient sign changes:** inspect model comparisons, exposure support, and influential observations; report the sensitivity rather than the preferred sign.
- **Country coefficients dominate the table:** they are nuisance parameters — retain them but present a focused precipitation-estimand table.
- **Quadratic term is significant:** do not interpret it in isolation; plot fitted values and contrasts at supported precipitation levels.
- **Residuals are temporally correlated:** default standard errors may be inappropriate; report the evidence rather than claiming independence.
- **Confounder unavailable:** do not silently replace it with a country indicator — record it as unmeasured and bound the causal claim.
- **High \(R^2\):** fit is not identification; the precipitation coefficient can remain causally biased regardless.
- **Script modifies integrated data:** stop — read the derived artifact and write results only, never overwrite `data/input/` or `data/derived/`.

---

## Completion checklist

- [ ] Question, contrast, outcome, timing, and estimand are stated, including the meaning of an additional 100 mm.
- [ ] The DAG distinguishes confounders, mediators, colliders, and proxies with justified arrows.
- [ ] Identification assumptions are assessed and exposure support is inspected by country and period.
- [ ] The regression sequence was specified before inspecting significance; estimates retain units and intervals.
- [ ] Nonlinear sensitivity uses contrasts, not one coefficient; residuals and influence are inspected.
- [ ] Statistical and identification uncertainty are kept separate; results are not selected by p-value.
- [ ] The conclusion states what is and is not causally supported, and code recreates all outputs.

---

## Reflect on the application

1. Which part of the provisional precipitation intervention remained most ambiguous?
2. Which causal arrows were supported by domain knowledge but not measured?
3. How did country and time terms change the estimate and its interpretation?
4. Which causal assumptions could not be assessed with residual diagnostics?
5. Which additional measurement or design would most strengthen the causal analysis?

---

## Further resources

- Miguel A. Hernán and James M. Robins, [Causal Inference: What If](https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/)
- Scott Cunningham, [Causal Inference: The Mixtape](https://mixtape.scunning.com/)
- [DAGitty](https://www.dagitty.net/)
- [R `lm` documentation](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/lm.html)
- [R diagnostic plots for linear models](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/plot.lm.html)
- [The Turing Way: Reproducible Research](https://book.the-turing-way.org/reproducible-research/)
