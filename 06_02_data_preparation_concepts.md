# 6.2) Understand data-preparation concepts

---

- Source: [06_02_data_preparation_concepts.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/06_02_data_preparation_concepts.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/06_02_data_preparation_concepts.md)
- Feedback: [Topic 06: Data Preparation](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/7)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Preparation and related activities](#preparation-and-related-activities)
- [Begin with an analysis contract](#begin-with-an-analysis-contract)
- [Observations, variables, and tidy structure](#observations-variables-and-tidy-structure)
- [Select and filter](#select-and-filter)
- [Rename, parse, and recode](#rename-parse-and-recode)
- [Reshape data](#reshape-data)
- [Convert units](#convert-units)
- [Missing values and structural absence](#missing-values-and-structural-absence)
- [Duplicates and repeated observations](#duplicates-and-repeated-observations)
- [Outliers and errors](#outliers-and-errors)
- [Derived variables and transformations](#derived-variables-and-transformations)
- [Data types and valid domains](#data-types-and-valid-domains)
- [Preparation audits](#preparation-audits)
- [Lineage and documentation](#lineage-and-documentation)
- [Preparation and data leakage](#preparation-and-data-leakage)
- [Idempotence and deterministic outputs](#idempotence-and-deterministic-outputs)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
- [Continue to Application](#continue-to-application)

---

## Learning objectives

After completing this page, you should be able to:

- distinguish validation, cleaning, integration, and preparation;
- specify a target population, grain, candidate key, variables, units, and missing-value policy;
- select, filter, parse, recode, reshape, convert, and derive values explicitly;
- distinguish missing values, duplicates, outliers, and errors;
- design checks that compare preparation inputs and outputs; and
- identify preprocessing operations that must be estimated from training data to prevent leakage.

---

## Place in the session

This is the **Concepts** part of the Data Preparation session:

~~~text
Motivation  →  Concepts  →  Application
                ↑
             this page
~~~

[Why prepare data?](06_01_data_preparation_motivation.md) explains why managed and integrated
evidence may not yet have the representation required for analysis. This page
provides a general vocabulary and decision model demonstrated in the
[maize-yield worked example](06_03_data_preparation_application.md).

For laboratory and field data, the analysis contract must preserve design
information such as biological versus technical replication, randomization,
blocks, sites, repeated units, collection occasions, and protocol deviations.
Those variables are not administrative clutter: they determine valid
summaries, models, and uncertainty.

Use these concepts as questions: What should one output row represent? Which
transformations change values or meaning, and what could be lost? Can every
output variable be traced to its inputs and formula?

---

## Preparation and related activities

These activities interact, but they answer different questions:

| Activity | Central question |
| --- | --- |
| Validation | Do data meet documented expectations? |
| Cleaning | Is a value demonstrably erroneous, and how should it be corrected? |
| Integration | How are compatible observations connected across sources? |
| Preparation | How should data be represented for the intended analysis? |
| Modeling preprocessing | Which transformations must be learned without seeing test outcomes? |

The same operation can play different roles: converting a provider's text
year to an integer is source normalization, centering a predictor using the
training mean is modeling preprocessing, and replacing a documented invalid
code is cleaning — the code alone does not establish the purpose. Validation
should generally precede transformation; a failed expectation should not be
hidden by a later filter, coercion, or aggregation.

---

## Begin with an analysis contract

An **analysis contract** describes the intended output before code is
written:

| Component | Question |
| --- | --- |
| Purpose, population, grain | Which task will use the output, which entities/periods does it represent, and what does one row mean? |
| Candidate key, variables | Which variables identify a row uniquely, and which source/derived variables are required? |
| Units, time | Which unit/scale and date, period, or reporting-year convention applies? |
| Missingness, ordering | Which missing states are possible, and is row order meaningful? |
| Output | Where is the artifact written and which process recreates it? |

For the maize project:

> One row represents one selected project country and year, containing maize
> yield, production, harvested area, and precipitation for the October–April
> season ending in that year.

The candidate key is:

~~~text
project_country_id + year
~~~

A contract is not permanent: version a new preparation rather than silently
changing an existing output's meaning.

---

## Observations, variables, and tidy structure

A common tidy-data convention is:

- each variable has its own column;
- each observation has its own row; and
- each type of observational unit has its own table.

This makes many operations predictable, but does not mean every source must
be stored in tidy form. For FAOSTAT, a provider-oriented long table preserves
element and unit as values:

| area | year | element | unit | value |
| --- | ---: | --- | --- | ---: |
| Zambia | 2022 | Yield | kg/ha | 2340 |
| Zambia | 2022 | Production | t | 3621000 |

For country-year analysis, yield and production can become separate variables:

| country | year | yield_kg_per_hectare | production_tonnes |
| --- | ---: | ---: | ---: |
| Zambia | 2022 | 2340 | 3621000 |

The important question is not whether a table looks tidy, but whether its
grain, key, variables, and relationships are explicit and appropriate.

---

## Select and filter

**Selection** chooses variables; **filtering** chooses observations. Before
filtering, state the inclusion rule; after filtering, record its effect on
row count, entities and periods, missingness, and the intended population.

Example:

~~~r
maize <- raw |>
  dplyr::filter(item == "Maize (corn)")
~~~

This is justified when the analysis contract concerns maize. A filter such as
<code>filter(!is.na(yield))</code> requires more care, since it changes the
population to complete observations and can introduce selection bias. Avoid
choosing rows by position; choose them by explicit identifiers, dates,
statuses, or values.

---

## Rename, parse, and recode

**Renaming** can improve consistency and readability, but must retain a
mapping to the source names. A good name indicates meaning and, for measures,
often the unit:

~~~text
Value  →  yield_tonnes_per_hectare
~~~

This mapping is valid only for rows already identified as the FAOSTAT Yield
element with unit <code>kg/ha</code> and after converting the values.

**Parsing** converts a representation into a data type — text to integer,
double, date, or categorical value. Check failed parses explicitly: a parser
that returns missing values may have encountered an invalid source value, an
unexpected locale, or an incomplete format.

**Recoding** maps values or classifications:

~~~r
measure <- dplyr::case_when(
  element == "Yield" & unit == "kg/ha" ~ "yield_kg_per_hectare",
  element == "Production" & unit == "t" ~ "production_tonnes",
  element == "Area harvested" & unit == "ha" ~ "harvested_area_hectares",
  TRUE ~ NA_character_
)
~~~

Every input value should map to an expected output or be reported unresolved;
a silent "other" fallback can conceal source changes.

---

## Reshape data

A **wide** transformation turns category values into columns. Before
widening, test that the intended identifiers plus the column-name variable
identify at most one value:

~~~text
country + year + measure
~~~

Then:

~~~r
panel <- tidy |>
  tidyr::pivot_wider(
    names_from = measure,
    values_from = value
  )
~~~

If <code>pivot_wider()</code> produces list columns or requests an
aggregation, stop and inspect the key. Choosing the first value or mean
merely to complete the operation can destroy information.

A **long** transformation turns several columns into a name-value pair,
useful when the same operation or visualization should apply to several
measures. Record which columns were gathered, the names and types of the new
variables, and whether the transformation is reversible. Reshaping should
change representation, not silently change grain through aggregation.

---

## Convert units

A unit conversion requires the source and target definitions and units, a
formula, the valid domain, dimensional consistency, and a check on
representative values. For maize yield:

~~~text
yield_tonnes_per_hectare = yield_kg_per_hectare / 1000
~~~

The denominator remains hectares; only kilograms convert to tonnes. A
recommended check:

~~~r
stopifnot(
  all.equal(
    yield_tonnes_per_hectare * 1000,
    yield_kg_per_hectare
  )
)
~~~

Do not mix unit conversion with a conceptual change: converting daily
precipitation to a growing-season total also requires temporal aggregation,
and must document the period and summary rule.

---

## Missing values and structural absence

A missing value can represent different states — not reported, not
applicable, invalid after parsing, not observed after a join, or
intentionally withheld — and each deserves its own representation rather than
being converted to zero or silently dropped.

Imputation creates estimated values and requires a method, assumptions, an
indicator of which values were imputed, and — when used for prediction —
training-only estimation. Compute missingness summaries before and after
preparation by important groups such as country and year.

---

## Duplicates and repeated observations

A **duplicate** is not defined merely by two identical-looking rows — it
depends on the expected grain and candidate key. Repeated records may be
legitimate when they differ by item, measurement method, source status or
revision, unit, spatial support, time period, or quality flag.

When a candidate key is duplicated, preserve the records, inspect the omitted
variables, consult source documentation, and justify any aggregation or
correction rather than calling <code>distinct()</code> without understanding
the key.

---

## Outliers and errors

An **outlier** is unusual relative to a distribution or model; an **error** is
a value known to be incorrect under justified evidence — not synonyms.
Investigate using provider flags, definitions, neighboring periods, and
domain knowledge, then retain and report, correct through a documented rule,
exclude while preserving the source, or mark unresolved. Deleting a point
because it changes a result is not a defensible preparation rule.

---

## Derived variables and transformations

A derived variable should have a precise name and definition, its input
variables and source artifacts, a formula or algorithm, a unit, a valid
domain, missingness behavior, and an interpretation and limitation.

The example project derives:

~~~text
log_yield = log(yield_tonnes_per_hectare)
~~~

The logarithm is defined only for positive yield; a safe transformation
returns missing for zero, negative, or missing inputs and records that rule.
It also changes interpretation — a one-unit change on the log scale is not a
one-tonne-per-hectare change — so the untransformed yield should be retained
for plots and interpretation on the original scale.

Other common derived variables include rates, proportions, differences,
rolling or grouped summaries, categorical bins, and interaction terms. Each
introduces assumptions and should be created as late as necessary for its
purpose.

---

## Data types and valid domains

A storage type does not fully specify a variable:

| Variable | Storage type | Domain |
| --- | --- | --- |
| Project country ID | Character | Nine reviewed codes |
| Year | Integer | 1990–2022 |
| Yield | Double | Non-negative, t/ha |
| Season start/end | Date | 1 October–30 April spanning the reporting year |

Validate domains after parsing and transformation: a numeric value can still
have an invalid unit or range, and a character variable can still contain an
unknown code. Dates need explicit formats and time semantics — the string
"2022" can mean a calendar year, reporting year, or season-ending year, and
its storage type alone does not resolve that meaning.

---

## Preparation audits

A preparation audit compares inputs, decisions, and outputs, for example:

| Check | Example expectation |
| --- | --- |
| Input identity | Expected path or checksum, unchanged after the run |
| Input key | No duplicate area-year-element-unit keys |
| Mappings | Every element-unit combination is recognized |
| Output key | Country-year is unique; row count matches expectation |
| Unit conversion | Converted values reproduce source values |
| Missingness and domain | Changes match documented rules (e.g. log yield missing only when yield is non-positive) |

Classify checks consistently as **pass** (expectation satisfied), **warning**
(review required but evidence remains usable), **failure** (a critical
contract is violated and output should not be written), or **unknown** (code
alone cannot decide). An audit should be machine-readable and summarized for
people, and should not rewrite the input to make its own checks pass.

---

## Lineage and documentation

The prepared dataset needs several complementary records: a **data
dictionary** defining every output variable, type, unit, role, allowed
values, and missingness; **human-readable documentation** explaining what the
dataset represents, how it was prepared, and its limitations; and
**provenance and lineage** — for example, a compact table mapping each output
variable to its source field and operation.

The script is executable evidence, but it is not a substitute for these
records.

---

## Preparation and data leakage

A transformation causes **data leakage** when it uses information unavailable
at prediction time, or lets test data influence model training — for example,
imputing or scaling with full-dataset statistics, or selecting variables
using test outcomes. Deterministic, source-defined operations (unit
conversion, explicit label mapping, parsing a documented date format) are
usually safe before the split; when uncertain, separate preparation into
those deterministic transformations and model-estimated preprocessing fit on
training data only. This boundary becomes central in the Predictive Modeling
session.

---

## Idempotence and deterministic outputs

A deterministic preparation produces the same output from the same inputs,
code, parameters, and environment. An **idempotent** workflow can be rerun
safely without accumulating duplicated rows or repeatedly transforming its
own output. Guidelines: read managed
inputs rather than a previous derived output unless the dependency is
explicit; overwrite generated outputs only after checks pass; avoid
current-date, random, or row-order dependence unless controlled; sort output
by explicit keys; and fail visibly on unresolved schemas or mappings.

Checksums can establish byte identity, but deterministic meaning also depends
on package versions, locale, numeric representation, and output ordering.

---

## Check your understanding

1. How do validation and preparation differ?
2. What information belongs in an analysis contract?
3. What should you inspect when a wide transformation produces multiple values per cell?
4. Why is missing not equivalent to zero?
5. When is a repeated row not a duplicate?
6. What evidence is needed before calling an outlier an error?
7. Which preprocessing operations must be fit using training data only?
8. What makes a preparation workflow deterministic and safe to rerun?

---

## Further resources

- [R for Data Science (2e): Data transform](https://r4ds.hadley.nz/data-transform.html)
  and [Data tidying](https://r4ds.hadley.nz/data-tidy.html)
- [The Turing Way: Data cleaning](https://book.the-turing-way.org/reproducible-research/rdm/rdm-cleaning/)
- [tidymodels: Preprocess your data with recipes](https://www.tidymodels.org/start/recipes/)
  and [Feature Engineering and Selection](https://bookdown.org/max/FES/)

---

## Continue to Application

Continue with [Prepare the maize country-year data](06_03_data_preparation_application.md). The
application states the preparation contract, inspects the managed FAOSTAT
input, reshapes its elements, converts and derives variables, validates the
country-year panel, integrates CHIRPS, and documents the resulting artifacts.
