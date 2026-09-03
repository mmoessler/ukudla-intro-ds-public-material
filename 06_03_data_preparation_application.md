# 6.3) Prepare the maize country-year data

---

- Last Update: 2026-08-21
- Source: [06_03_data_preparation_application.md](/learning-modules/intro-ds-module/06_03_data_preparation_application.md)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Scenario and deliverables](#scenario-and-deliverables)
- [Before you begin](#before-you-begin)
- [1. State the preparation contract](#1-state-the-preparation-contract)
- [2. Inspect the managed input](#2-inspect-the-managed-input)
- [3. Validate mappings before reshaping](#3-validate-mappings-before-reshaping)
- [4. Create the country-year maize panel](#4-create-the-country-year-maize-panel)
- [5. Verify unit conversion and derived variables](#5-verify-unit-conversion-and-derived-variables)
  - [Yield conversion](#yield-conversion)
  - [Log yield](#log-yield)
- [6. Integrate the prepared panel with CHIRPS](#6-integrate-the-prepared-panel-with-chirps)
- [7. Audit the prepared artifacts](#7-audit-the-prepared-artifacts)
- [8. Document preparation and lineage](#8-document-preparation-and-lineage)
- [Preparation before predictive modeling](#preparation-before-predictive-modeling)
- [Troubleshooting](#troubleshooting)
- [Completion checklist](#completion-checklist)
- [Reflect on the application](#reflect-on-the-application)
- [Further resources](#further-resources)

---

## Learning objectives

After completing this exercise, you should be able to:

- state the target population, grain, key, variables, units, and missing-value policy;
- validate element-unit mappings and source-key uniqueness before reshaping;
- reshape the long FAOSTAT values into a country-year panel;
- verify a unit conversion and a log transformation; and
- identify the documentation required for intermediate and final derived data.

---

## Place in the session

This is the **Application** part of the Data Preparation session:

~~~text
Motivation  →  Concepts  →  Application
                              ↑
                           this page
~~~

Before beginning, review [Why prepare data?](06_01_data_preparation_motivation.md) and [Understand
data-preparation concepts](06_02_data_preparation_concepts.md).

The preceding sessions established managed FAOSTAT and CHIRPS inputs and an
audited integration workflow. This exercise focuses on the transformations
that create the country-year representation required by later visualization,
description, and modeling.

Do not continue through a failed key, mapping, or coverage expectation only
to obtain the expected output filename.

---

## Scenario and deliverables

The managed FAOSTAT teaching input stores three maize elements in long form
(Yield in <code>kg/ha</code>, Production in <code>t</code>, Area harvested in
<code>ha</code>). The project needs one row per country and year with
separate columns for those measures, plus stable project identifiers and
growing-season precipitation from the preceding integration topic.

The workflow creates:

~~~text
data/derived/maize-yield-panel.csv
data/derived/maize-yield-with-precipitation.csv
results/tables/data-integration-audit.csv
~~~

The maize panel is an intermediate derived artifact; the integrated table is
the current input to exploration and modeling.

For this exercise, the preparation evidence should establish the input and
output grain and keys, element-unit mappings, row-count and coverage
expectations, unit conversion and derived-variable domain, missingness
behavior, unchanged managed inputs, and lineage from source fields to output
variables.

---

## Before you begin

Work from the standalone <code>maize-yield-project</code> repository. Confirm
that the working tree is in the expected state:

~~~bash
pwd
git status --short
~~~

Restore and verify the recorded environment:

~~~bash
Rscript scripts/setup.R
~~~

Confirm that the fixed inputs exist:

~~~bash
ls -lh \
  data/input/faostat-maize-yield-sample.csv \
  data/input/chirps-growing-season-precipitation.csv
~~~

The student workflow uses these tracked snapshots and requires no network
connection; do not run the acquisition scripts for this exercise.

Relevant files include:

| Role | File |
| --- | --- |
| Managed FAOSTAT input | <code>data/input/faostat-maize-yield-sample.csv</code> |
| FAOSTAT dictionary | <code>metadata/faostat-maize-yield-data-dictionary.csv</code> |
| FAOSTAT flag definitions | <code>metadata/faostat-flag-code-list.csv</code> |
| Provenance | <code>metadata/provenance.yml</code> |
| Preparation script | <code>scripts/prepare-maize-data.R</code> |
| Prepared maize panel | <code>data/derived/maize-yield-panel.csv</code> |
| Integration script | <code>scripts/integrate-data.R</code> |
| Final integrated dictionary | <code>metadata/maize-yield-with-precipitation-data-dictionary.csv</code> |

The <code>data/derived/</code> directory contains generated outputs and is
ignored by Git. Recreate its contents through scripts rather than editing them.

---

## 1. State the preparation contract

Complete the following before running the script:

~~~text
Purpose:
Input artifact:
Input grain:
Input candidate key:
Output artifact:
Output population:
Output grain:
Output candidate key:
Retained variables:
Source and target units:
Derived variables:
Missing-value policy:
Expected row count:
~~~

For this project:

| Component | Contract |
| --- | --- |
| Purpose | Country-year exploration and modeling of maize yield |
| Input | Fixed FAOSTAT maize teaching sample |
| Input grain | One area-item-element-year-unit observation |
| Input key | <code>area + item + element + year + unit</code> |
| Output | Prepared maize country-year panel |
| Population | Nine selected countries, 1990–2022 |
| Output grain | One country and year |
| Output key | <code>country + year</code> |
| Expected rows | 9 countries × 33 years = 297 |
| Yield unit | Convert <code>kg/ha</code> to <code>t/ha</code> |
| Derived variable | Natural log of positive yield |
| Missingness | Preserve missing measures; never replace with zero |

Predict which source columns will remain and which will become output column
names and units.

---

## 2. Inspect the managed input

Read without modifying:

~~~r
library(dplyr)
library(readr)

raw <- read_csv(
  "data/input/faostat-maize-yield-sample.csv",
  show_col_types = FALSE
)

glimpse(raw)
nrow(raw)
names(raw)
~~~

Inspect coverage and classifications:

~~~r
raw |>
  count(area, name = "rows")

raw |>
  distinct(item)

raw |>
  distinct(element, unit) |>
  arrange(element, unit)

range(raw$year)
~~~

Confirm nine areas, one item (<code>Maize (corn)</code>), years 1990–2022,
three documented element-unit combinations, and 891 rows before reshaping.
Read the data dictionary and flag code list: a variable name alone is
insufficient to interpret <code>value</code>, since its meaning and unit
depend on <code>element</code> and <code>unit</code>.

---

## 3. Validate mappings before reshaping

The preparation script expects:

~~~r
expected_element_units <- c(
  "Area harvested|ha",
  "Production|t",
  "Yield|kg/ha"
)
~~~

Compare observed pairs:

~~~r
observed_element_units <- raw |>
  distinct(element, unit) |>
  transmute(pair = paste(element, unit, sep = "|")) |>
  pull(pair)

setequal(observed_element_units, expected_element_units)
~~~

Test the source candidate key:

~~~r
candidate_key <- c("area", "item", "element", "year", "unit")

duplicate_keys <- raw |>
  count(across(all_of(candidate_key))) |>
  filter(n > 1)

duplicate_keys
~~~

Expected result: no duplicate candidate keys.

The mapping from provider elements to project measures is:

| Source element | Source unit | Project measure |
| --- | --- | --- |
| Yield | kg/ha | <code>yield_kg_per_hectare</code> |
| Production | t | <code>production_tonnes</code> |
| Area harvested | ha | <code>harvested_area_hectares</code> |

Every observed pair must map exactly once; do not use a catch-all label.

---

## 4. Create the country-year maize panel

Run the existing project script:

~~~bash
Rscript scripts/prepare-maize-data.R
~~~

The script first creates a normalized long representation:

~~~r
tidy <- raw |>
  filter(item == "Maize (corn)") |>
  transmute(
    country = area,
    year = as.integer(year),
    measure = case_when(
      element == "Yield" & unit == "kg/ha" ~ "yield_kg_per_hectare",
      element == "Area harvested" & unit == "ha" ~ "harvested_area_hectares",
      element == "Production" & unit == "t" ~ "production_tonnes",
      TRUE ~ NA_character_
    ),
    value = as.numeric(value)
  )
~~~

It then reshapes:

~~~r
panel <- tidy |>
  tidyr::pivot_wider(
    names_from = measure,
    values_from = value
  ) |>
  arrange(country, year)
~~~

Inspect the output:

~~~r
panel <- read_csv(
  "data/derived/maize-yield-panel.csv",
  show_col_types = FALSE
)

glimpse(panel)
nrow(panel)
~~~

Test the output key:

~~~r
panel |>
  count(country, year) |>
  filter(n != 1)
~~~

Expected result: 297 rows and no duplicate country-year key.

If widening produces list columns or multiple values per cell, return to the
input key. Do not choose an arbitrary aggregation function to force a scalar
output.

---

## 5. Verify unit conversion and derived variables

### Yield conversion

The project converts:

~~~text
yield_tonnes_per_hectare = yield_kg_per_hectare / 1000
~~~

Verify the conversion against the source values:

~~~r
source_yield <- raw |>
  filter(element == "Yield", unit == "kg/ha") |>
  transmute(
    country = area,
    year,
    source_yield_kg_per_hectare = value
  )

conversion_check <- panel |>
  left_join(source_yield, by = c("country", "year")) |>
  mutate(
    reconstructed_kg_per_hectare =
      yield_tonnes_per_hectare * 1000,
    difference =
      reconstructed_kg_per_hectare -
      source_yield_kg_per_hectare
  )

max(abs(conversion_check$difference), na.rm = TRUE)
~~~

Expected result: zero or negligible floating-point difference.

### Log yield

The project derives:

~~~text
log_yield = log(yield_tonnes_per_hectare)
~~~

Check its domain and behavior:

~~~r
panel |>
  summarise(
    non_positive_yield =
      sum(yield_tonnes_per_hectare <= 0, na.rm = TRUE),
    missing_log_for_positive =
      sum(
        is.na(log_yield) &
        yield_tonnes_per_hectare > 0,
        na.rm = TRUE
      ),
    finite_log_values =
      all(is.finite(log_yield[!is.na(log_yield)]))
  )
~~~

The log is undefined for non-positive values. The original yield remains in the
panel so results can be interpreted on the physical scale.

---

## 6. Integrate the prepared panel with CHIRPS

Preparation and integration have a dependency:

~~~text
prepared maize panel
        +
CHIRPS country-season snapshot
        +
project country crosswalk
        ↓
integrated country-year table
~~~

Run:

~~~bash
Rscript scripts/integrate-data.R
~~~

The integration script:

- maps <code>country</code> to <code>project_country_id</code>;
- validates both country-year keys;
- left-joins precipitation on project identifier and year;
- preserves 297 maize rows;
- records unmatched keys and missing precipitation; and
- writes an integration audit.

Inspect:

~~~r
integrated <- read_csv(
  "data/derived/maize-yield-with-precipitation.csv",
  show_col_types = FALSE
)

glimpse(integrated)
nrow(integrated)

integrated |>
  count(project_country_id, year) |>
  filter(n != 1)
~~~

The final table has one row per project country and year. The CHIRPS season
ending in a year is aligned to the FAOSTAT observation for that year.

---

## 7. Audit the prepared artifacts

A dedicated preparation audit is not yet stored by the example project. For
this exercise, assemble these checks in code or a table:

| Check | Expectation |
| --- | --- |
| Managed input rows | 891 |
| Managed input key duplicates | 0 |
| Recognized element-unit pairs | Exactly 3 expected pairs |
| Prepared panel rows | 297 |
| Prepared output key duplicates | 0 |
| Country coverage | 9 |
| Year coverage | 1990–2022 |
| Unit-conversion discrepancy | 0 within numeric tolerance |
| Missing log for positive yield | 0 |
| Non-finite retained log values | 0 |
| Managed input checksum | Matches provenance before and after |
| Integrated output rows | 297 |
| Integrated output key duplicates | 0 |

A possible machine-readable structure is:

~~~text
check,expectation,observed,status
input-rows,891,891,pass
input-key-duplicates,0,0,pass
prepared-rows,297,297,pass
...
~~~

Do not hard-code a passing status independently of the observed result. A
critical failure should prevent the output from being treated as ready.

This audit is a recommended future improvement to the example project. Until
it is implemented there, the existing script checks and integration audit are
the executable evidence.

---

## 8. Document preparation and lineage

The final integrated artifact already has:

- a human-readable page:
  <code>docs/data/maize-yield-with-precipitation.md</code>;
- a data dictionary:
  <code>metadata/maize-yield-with-precipitation-data-dictionary.csv</code>;
- lineage in <code>metadata/provenance.yml</code>; and
- an integration report and audit.

The intermediate <code>maize-yield-panel.csv</code> is recorded in provenance
but currently lacks its own data dictionary and dataset page. A complete
implementation should consider adding:

~~~text
metadata/maize-yield-panel-data-dictionary.csv
docs/data/maize-yield-panel.md
results/tables/data-preparation-audit.csv
docs/data-preparation.md
~~~

Avoid duplicating authoritative facts: Markdown explains purpose and
limitations, CSV defines output variables, YAML records artifact history,
scripts execute transformations, and audit tables record observed checks.

A lineage table for the maize panel should include:

| Output variable | Source field | Operation |
| --- | --- | --- |
| <code>country</code> | <code>area</code> | Rename retained label |
| <code>year</code> | <code>year</code> | Parse as integer |
| <code>yield_tonnes_per_hectare</code> | Yield <code>value</code> in kg/ha | Divide by 1000 |
| <code>production_tonnes</code> | Production <code>value</code> in t | Reshape |
| <code>harvested_area_hectares</code> | Area harvested <code>value</code> in ha | Reshape |
| <code>log_yield</code> | Prepared yield | Natural log for positive values |

---

## Preparation before predictive modeling

The current transformations are based on fixed source definitions: element-unit
mapping, deterministic reshaping, unit conversion, country crosswalk, temporal
alignment, and a log formula with a fixed mathematical definition.

Later modeling steps may introduce transformations learned from observed data
— mean/median imputation, centering and scaling, feature selection, or
data-dependent bins — that must be estimated using training data only and
applied unchanged to the test period. Do not add such operations to a general
preparation script that runs on the complete dataset.

---

## Troubleshooting

| Symptom | Response |
| --- | --- |
| The input file is missing | Run <code>git status</code> and confirm the tracked teaching input was checked out; do not run a provider download automatically. |
| Required columns are missing | Compare the file with its dictionary and provenance — a changed schema may mean the wrong artifact is present. |
| An element-unit combination is unexpected | Do not map by element alone; inspect the pair and provider metadata before updating the contract. |
| The source key is duplicated | Do not call <code>distinct()</code> automatically; determine whether the key omits a classification, unit, or revision dimension. |
| Widening creates multiple values | The proposed target cell is not unique; return to the input grain and mapping. |
| The panel has fewer than 297 rows | Inspect missing country-year-element combinations; do not create rows or fill values until the absence is understood. |
| The log contains infinite values | Inspect zero or negative yield; the safe transformation should not retain infinite values. |
| A rerun changes managed inputs | Stop — preparation must write only derived artifacts. Restore the input through version control and inspect the script paths. |

---

## Completion checklist

- [ ] The purpose, population, grain, key, variables, units, and missingness policy are stated.
- [ ] The managed FAOSTAT input remains unchanged.
- [ ] Required columns and element-unit combinations are validated.
- [ ] The source candidate key is unique.
- [ ] The long-to-wide transformation preserves the intended country-year grain.
- [ ] Yield conversion is verified against source values.
- [ ] Log yield is finite and missing only under the documented rule.
- [ ] The prepared panel has 297 unique country-year rows.
- [ ] Integration preserves 297 unique project-country-year rows.
- [ ] Missingness and coverage changes are explained.
- [ ] Output variables can be traced to inputs and formulas.
- [ ] Preparation checks are recorded or identified as a current implementation gap.
- [ ] Data-dependent modeling preprocessing is deferred until after the train/test split.

---

## Reflect on the application

1. Why is the long FAOSTAT input useful even though the analysis uses a wide panel?
2. Which variables define a unique value before widening?
3. Why is division by 1000 a unit conversion rather than an arbitrary scaling?
4. Why retain yield on the original physical scale after creating log yield?
5. Which preparation steps must occur before CHIRPS integration, and which could occur after?
6. Which future preprocessing operations could leak test information?

---

## Further resources

- [R for Data Science (2e): Data transform](https://r4ds.hadley.nz/data-transform.html)
- [pivot_wider — tidyr reference](https://tidyr.tidyverse.org/reference/pivot_wider.html)
- [Mutate joins — dplyr reference](https://dplyr.tidyverse.org/reference/mutate-joins.html)
