# Conduct a causal analysis of maize yield

---

- Last Update: 2026-08-25
- Source: [09_explanatory_analysis_application.md](/learning-modules/intro-ds-module/09_explanatory_analysis_application.md)
- Estimated completion time: 6–8 hours
- Independent extension: 2–3 hours
- Prerequisites: Motivation and Concepts pages; completed Descriptive Data Analysis workflow
- Required output: causal question, DAG, identification assessment, explanatory-modeling script, model and diagnostic tables, and bounded conclusion

---

## Outline

- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Scenario and deliverables](#scenario-and-deliverables)
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
- encode causal assumptions in a directed acyclic graph;
- assess consistency, exchangeability, positivity, interference, measurement, and selection;
- distinguish a proposed adjustment set from variables included only for prediction or convenience;
- fit and compare transparent linear-regression specifications in R;
- scale and interpret a precipitation coefficient per 100 mm;
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
reasoning](09_explanatory_analysis_motivation.md) and [Understand
explanatory-modeling concepts](09_explanatory_analysis_concepts.md).

Descriptive Data Analysis established coverage, country and period summaries,
association scopes, and evidence relevant to stationarity. This exercise uses
that evidence to design and challenge a causal analysis. It does not overwrite
data or reuse the predictive model as if prediction and causal explanation
were the same task.

---

## Scenario and deliverables

The project team asks:

> What is the causal effect of growing-season precipitation on national maize
> yield in the selected Southern African countries?

Your task is to turn this broad question into an auditable analysis and decide
whether the available data identify the intended effect.

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

The causal-model document records the question, estimand, diagram, variable
roles, assumptions, and identification judgment. The script produces
reproducible statistical evidence. The conclusion must distinguish what the
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
| Country and period descriptions | `results/tables/maize-yield-period-summary.csv` |
| Association by scope | `results/tables/yield-precipitation-association.csv` |
| Stability evidence | `results/tables/stationarity-diagnostic.csv` |
| Modeling handoff | `results/descriptive-modeling-handoff.md` |

The input contains 297 country-years, not randomized rainfall treatments,
field-level exposures, or measurements of every common cause.

---

## 1. Define the causal question and estimand

Create `docs/causal-model.md` and complete:

| Element | Provisional definition |
| --- | --- |
| Target population | Country-years for nine selected countries, 1990–2022 |
| Unit | One country-year |
| Exposure | October-April country-area CHIRPS precipitation total |
| Contrast | 100 mm higher versus the observed reference level |
| Outcome | National maize yield in tonnes per hectare for the ending year |
| Time zero | Beginning of the October-April growing-season window |
| Follow-up | Yield reported for the corresponding ending year |
| Estimand | Average difference in potential national yield under the two exposure conditions |

Then challenge every row. In particular, explain why “100 mm higher” does not
specify rainfall timing, intensity, location, or mechanism. Decide whether the
contrast should be treated as a provisional scientific target rather than a
fully well-defined intervention.

Write the potential-outcomes expression:

\[
E\left[Y(P+100)-Y(P)\right]
\]

State whether a constant shift is meaningful throughout the observed range.
Do not change the estimand after inspecting which model is significant.

---

## 2. Draw the causal diagram

Begin with concepts, not only available columns. Include at least:

- precipitation amount, timing, and intensity;
- temperature and other seasonal weather;
- national maize yield;
- irrigation and water access;
- soils and maize-growing area;
- seed varieties, fertilizer, labor, and management;
- pests and disease;
- policy, markets, conflict, and reporting practices;
- country context and calendar time; and
- measurement and selection processes.

For every arrow, write one sentence explaining the assumed causal mechanism
and temporal order. Mark whether each node is measured, proxied, or unmeasured.

Use a simplified diagram in the report, while retaining the full inventory in
`docs/causal-model.md`. A possible starting structure is:

~~~text
seasonal weather ───► precipitation ───► maize yield
       │                                      ▲
       └──────────────────────────────────────┘

country/time context ─► precipitation
          │                    │
          └───────────────────►yield

irrigation, soils, inputs and management ────►yield
~~~

This is not a finished adjustment strategy. Discuss which omitted factors may
also cause precipitation exposure or influence its measurement. Do not use
the diagram merely to justify variables already in the table.

---

## 3. Assess identification before estimation

Add an identification table:

| Requirement | Project evidence | Judgment | Consequence |
| --- | --- | --- | --- |
| Consistency | Seasonal total; timing and spatial versions hidden | Doubtful | The 100 mm contrast is ambiguous |
| Exchangeability | Country and year observed; important weather and management causes omitted | Not established | Adjusted coefficient may remain confounded |
| Positivity | 33 observations per country; country ranges differ | Must inspect | Some contrasts may require extrapolation |
| No interference | Country-years connected by markets, water, and regional shocks | Uncertain | Unit-level treatment representation may be incomplete |
| Measurement | National yield and country-area rainfall | Limited | Exposure-outcome alignment is imperfect |
| Selection | Nine selected countries with complete teaching data | Limited target | Do not generalize automatically |

Do not write “assumption met” merely because it cannot be tested. State what
evidence would be needed. Decide before modeling whether the dataset plausibly
identifies the estimand. The core expectation is that it does not fully do so.

The statistical models remain useful for quantifying how associations change
under selected adjustments and for demonstrating the consequences of the
design limitations.

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

Compare country ranges and distributions. A model with country indicators
primarily uses within-country variation. Identify where a 100 mm contrast lies
inside observed support and where it implies extrapolation. Range overlap is a
necessary descriptive check, not proof of conditional positivity.

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
| Unadjusted | All country-years | Reproduce pooled linear association | Country and time differences |
| Country | Within-country, common slope | Account for stable country levels | Time-varying confounding |
| Time | Conditional on common trend | Account for common linear change | Country differences |
| Country + time | Within-country around common trend | Main adjusted association | Omitted changing causes and dependence |
| Nonlinear | Curved conditional association | Functional-form sensitivity | Same identification limitations |

Country and time terms are not declared a sufficient adjustment set. They are
transparent, available proxies used to show how model interpretation changes.

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

Interpret each estimate in tonnes per hectare per 100 mm and name its
conditioning variables. Compare direction and magnitude across models before
discussing p-values.

The intervals use default `lm` assumptions, including an error structure that
does not represent all within-country temporal dependence. They quantify
model-based sampling uncertainty, not uncertainty from confounding,
measurement, intervention ambiguity, or specification choice.

Do not interpret the linear term from the quadratic model alone. In a quadratic
model the modeled difference varies with baseline precipitation; inspect
predicted contrasts across supported values.

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

Inspect, do not merely calculate:

- residuals versus fitted values;
- residual distribution and scale-location pattern;
- observations with high leverage or Cook's distance;
- residuals over time within countries;
- lag-one residual correlation by country; and
- precipitation support behind fitted linear and nonlinear patterns.

Create `explanatory-residual-dependence.csv` with lag-one pair counts and
correlations for the main `country_time` model. Verify consecutive years before
forming pairs.

Ask:

1. Does adjustment substantially change the precipitation coefficient?
2. Does a linear form miss a visible curve?
3. Are conclusions driven by a small number of country-years?
4. Do residuals retain country-specific temporal structure?
5. Which findings are stable across planned specifications?

Diagnostics can reveal statistical misspecification. They cannot establish
exchangeability or correct measurement.

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

- the estimated conditional association per 100 mm from every planned linear model;
- how estimates change with country and time terms;
- the nonlinear sensitivity result;
- relevant overlap, influence, residual, and temporal warnings;
- which identification assumptions are doubtful or unassessable;
- why default confidence intervals do not include identification uncertainty;
- an explicit statement that correlation and adjusted regression are not automatically causal; and
- concrete data or design improvements.

A defensible conclusion may be:

> The models quantify associations between country-area growing-season
> precipitation and national maize yield conditional on selected country and
> time terms. Because the exposure is not a well-defined intervention and
> important time-varying common causes and measurement limitations remain, the
> available data do not identify the proposed causal effect.

Render `reports/explanatory-modeling.qmd` so the causal question, diagram,
identification table, estimates, diagnostics, and conclusion remain linked to
the reproducible artifacts.

---

## Independent extension

Choose one extension and document its causal purpose and limitations.

### Option A: Alternative exposure representation

Compare the seasonal total with dry-day counts, rainfall intensity, or
sub-season periods if defensible data are available. Explain how this improves
consistency and whether it creates multiple-comparison concerns.

### Option B: Country-specific associations

Add precipitation-by-country interactions. Present joint country-specific
contrasts and explain the small sample size and increased uncertainty.

### Option C: Negative-control reasoning

Propose an exposure or outcome that should not be causally affected but may
share bias mechanisms. Explain what a detected association would reveal. Do
not invent a negative control solely from available variables.

### Option D: Stronger research design

Sketch a field experiment, irrigation intervention, natural experiment, or
subnational panel that could address the question more credibly. State its
assignment mechanism, estimand, measurements, and ethical or practical limits.

---

## Troubleshooting

### The precipitation coefficient changes sign

Inspect which comparison each model makes, country and time structure,
exposure support, and influential observations. Report the sensitivity rather
than selecting the preferred sign.

### Country coefficients dominate the table

They are nuisance parameters for this question. Retain them in the fitted
model but present a focused table of the precipitation estimand and model
specification.

### The quadratic term is significant

Do not interpret one polynomial coefficient in isolation. Plot fitted values
and calculate contrasts at supported precipitation levels. Identification
limitations remain unchanged.

### Residuals are temporally correlated

Default standard errors may be inappropriate and the model may omit temporal
structure. Report the evidence and consider a justified error model or more
advanced panel method; do not claim independence.

### A confounder is unavailable

Do not replace it silently with a country indicator. Record it as unmeasured,
explain the likely direction or ambiguity of bias, and bound the causal claim.

### The model has a high \(R^2\)

Fit is not identification. Country and time terms can explain outcome
variation while the precipitation coefficient remains causally biased.

### The script modifies the integrated data

Stop. Explanatory modeling reads the derived artifact and writes results. It
must not overwrite `data/input/` or `data/derived/`.

---

## Completion checklist

- [ ] Target population, unit, exposure, contrast, outcome, timing, and estimand are stated.
- [ ] Exposure versions and the meaning of an additional 100 mm are discussed.
- [ ] The DAG includes measured and unmeasured causes with justified arrows.
- [ ] Confounders, mediators, colliders, and proxies are distinguished.
- [ ] Consistency, exchangeability, positivity, interference, measurement, and selection are assessed.
- [ ] Exposure support is inspected by country and period.
- [ ] The regression sequence was specified before inspecting significance.
- [ ] Precipitation estimates retain units, sample sizes, intervals, and conditioning sets.
- [ ] Nonlinear sensitivity is interpreted through contrasts, not one coefficient.
- [ ] Residuals, variance, influence, and temporal dependence are inspected.
- [ ] Statistical uncertainty is separated from identification uncertainty.
- [ ] Results are not selected according to p-value or preferred direction.
- [ ] The conclusion states what is and is not causally supported.
- [ ] Code recreates all generated tables and report inputs.
- [ ] Managed and derived input data remain unchanged.

---

## Reflect on the application

1. Which part of the provisional precipitation intervention remained most ambiguous?
2. Which causal arrows were supported by domain knowledge but not measured?
3. Which adjustment variables were genuine confounder proxies, and which served other roles?
4. Where did a 100 mm contrast require extrapolation?
5. How did country and time terms change the estimate and its interpretation?
6. Which model diagnostics changed your statistical specification?
7. Which causal assumptions could not be assessed with residual diagnostics?
8. How did temporal dependence affect confidence in default uncertainty estimates?
9. What is the strongest interpretation supported by the current data?
10. Which additional measurement or design would most strengthen the causal analysis?

---

## Further resources

- Miguel A. Hernán and James M. Robins, [Causal Inference: What If](https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/)
- Scott Cunningham, [Causal Inference: The Mixtape](https://mixtape.scunning.com/)
- [DAGitty](https://www.dagitty.net/)
- [R `lm` documentation](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/lm.html)
- [R diagnostic plots for linear models](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/plot.lm.html)
- [The Turing Way: Reproducible Research](https://book.the-turing-way.org/reproducible-research/)
