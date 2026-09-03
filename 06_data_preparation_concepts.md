# Understand data-preparation concepts

---

- Last Update: 2026-08-21
- Source: [06_data_preparation_concepts.md](/learning-modules/intro-ds-module/06_data_preparation_concepts.md)

---

## Outline

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
- explain tidy observations and variables without treating tidy data as a universal final form;
- select, filter, parse, recode, reshape, convert, and derive values explicitly;
- distinguish missing values, duplicates, repeated observations, outliers, and errors;
- design checks that compare preparation inputs and outputs;
- document an analysis-ready artifact and its lineage; and
- identify preprocessing operations that must be estimated from training data to prevent leakage.

---

## Place in the session

This is the **Concepts** part of the Data Preparation session:

~~~text
Motivation  →  Concepts  →  Application
                ↑
             this page
~~~

[Why prepare data?](06_data_preparation_motivation.md) explains why managed and integrated
evidence may not yet have the representation required for analysis. This page
provides the vocabulary and decision model used in [the maize preparation
application](06_data_preparation_application.md).

Use these concepts as questions:

- What should one output row represent?
- Which variables should identify it?
- Which population and periods should remain?
- Which transformations change values, structure, or meaning?
- Which information could be lost?
- Which expectations can be checked before writing the output?
- Can every output variable be traced to its inputs and formula?
- Does any operation use information that should be unavailable at prediction time?

---

## Preparation and related activities

These activities interact, but they answer different questions:

| Activity | Central question | Typical evidence |
| --- | --- | --- |
| Validation | Do data meet documented expectations? | Checks, observed values, and findings |
| Cleaning | Is a value demonstrably erroneous, and how should it be corrected? | Justification and correction rule |
| Integration | How are compatible observations connected across sources? | Crosswalk, join contract, and audit |
| Preparation | How should data be represented for the intended analysis? | Transformation contract, script, and derived artifact |
| Modeling preprocessing | Which transformations must be learned without seeing test outcomes? | Training-fitted preprocessing object or recipe |

The same operation can play different roles. Converting a provider's text year
to an integer may be source normalization. Centering a predictor using the
training mean is modeling preprocessing. Replacing a documented invalid code
may be cleaning. The code alone does not establish the purpose or
justification.

Validation should generally precede transformation. A failed expectation
should not be hidden by a later filter, coercion, or aggregation.

---

## Begin with an analysis contract

An **analysis contract** describes the intended output before code is written.

Record:

| Component | Question |
| --- | --- |
| Purpose | Which analysis or communication task will use the output? |
| Population | Which entities and periods should the output represent? |
| Grain | What does one row represent? |
| Candidate key | Which variables should identify one row uniquely? |
| Variables | Which source and derived variables are required? |
| Units | Which unit and scale should each measure use? |
| Time | Which date, period, season, or reporting-year convention applies? |
| Missingness | Which missing states are possible and how are they represented? |
| Ordering | Is row order meaningful or only for reproducible display? |
| Output | Where is the artifact written and which process recreates it? |

For the maize project:

> One row represents one selected project country and year, containing maize
> yield, production, harvested area, and precipitation for the October–April
> season ending in that year.

The candidate key is:

~~~text
project_country_id + year
~~~

A contract is not permanent. If the analytical purpose changes, create or
version a new preparation rather than silently changing the meaning of an
existing output.

---

## Observations, variables, and tidy structure

A common tidy-data convention is:

- each variable has its own column;
- each observation has its own row; and
- each type of observational unit has its own table.

This convention makes many analytical operations predictable, but it does not
mean that every source must be stored in tidy form or that one shape suits
every task.

For FAOSTAT, a provider-oriented long table can preserve element and unit as
values:

| area | year | element | unit | value |
| --- | ---: | --- | --- | ---: |
| Zambia | 2022 | Yield | kg/ha | 2340 |
| Zambia | 2022 | Production | t | 3621000 |

For country-year analysis, yield and production can become separate variables:

| country | year | yield_kg_per_hectare | production_tonnes |
| --- | ---: | ---: | ---: |
| Zambia | 2022 | 2340 | 3621000 |

The important question is not whether a table looks tidy. It is whether its
grain, key, variables, and relationships are explicit and appropriate.

---

## Select and filter

**Selection** chooses variables. **Filtering** chooses observations.

Before filtering, state the inclusion rule. After filtering, record its effect
on:

- row count;
- entities and periods;
- missingness;
- groups represented;
- minimum and maximum values; and
- the intended population.

Example:

~~~r
maize <- raw |>
  dplyr::filter(item == "Maize (corn)")
~~~

This is justified when the analysis contract concerns maize. A filter such as
<code>filter(!is.na(yield))</code> requires more care because it changes the
population to complete observations and can introduce selection bias.

Avoid choosing rows by their position unless the ordering itself is a
documented part of the source. Choose them by explicit identifiers, dates,
statuses, or values.

---

## Rename, parse, and recode

### Rename

Project names can improve consistency and readability, but retain a mapping to
the source names.

A good name should indicate meaning and, for measures, often the unit:

~~~text
Value  →  yield_tonnes_per_hectare
~~~

The mapping is valid only for rows already identified as the FAOSTAT Yield
element with unit <code>kg/ha</code> and after converting the values.

### Parse

Parsing converts a representation into a data type:

- text to integer or double;
- text to date or date-time;
- text to logical or categorical values.

Check failed parses explicitly. A parser that returns missing values may have
encountered an invalid source value, an unexpected locale, or an incomplete
format.

### Recode

Recoding maps values or classifications:

~~~r
measure <- dplyr::case_when(
  element == "Yield" & unit == "kg/ha" ~ "yield_kg_per_hectare",
  element == "Production" & unit == "t" ~ "production_tonnes",
  element == "Area harvested" & unit == "ha" ~ "harvested_area_hectares",
  TRUE ~ NA_character_
)
~~~

Every input value should map to an expected output or be reported as unresolved.
A fallback that silently groups unknown values into "other" can conceal source
changes.

---

## Reshape data

### Long to wide

A wide transformation turns category values into columns.

Before widening, test that the intended identifiers plus the column-name
variable identify at most one value:

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

If <code>pivot_wider()</code> produces list columns or requests an aggregation,
stop and inspect the key. Choosing the first value or mean merely to complete
the operation can destroy information.

### Wide to long

A long transformation turns several columns into a name-value pair. It is
useful when the same operation or visualization should apply to several
measures.

Record:

- which columns were gathered;
- the names and types of the new variables;
- whether units remain distinguishable; and
- whether the transformation is reversible.

Reshaping should change representation, not silently change grain through
aggregation.

---

## Convert units

A unit conversion requires:

- the source definition and unit;
- the target definition and unit;
- a formula;
- the valid domain;
- dimensional consistency; and
- a check on representative values.

For maize yield:

~~~text
yield_tonnes_per_hectare = yield_kg_per_hectare / 1000
~~~

The denominator remains hectares. Only kilograms are converted to tonnes.

Recommended checks include:

~~~r
stopifnot(
  all.equal(
    yield_tonnes_per_hectare * 1000,
    yield_kg_per_hectare
  )
)
~~~

Do not mix unit conversion with a conceptual change. Converting daily
precipitation to a growing-season total requires temporal aggregation as well
as units and must document the period and summary rule.

---

## Missing values and structural absence

A missing value can represent different states:

| State | Example | Possible representation |
| --- | --- | --- |
| Not reported | Provider supplied no yield | <code>NA</code> plus source context |
| Not applicable | A measure does not apply to an entity | Explicit status or structural absence |
| Invalid | Parsing or validation failed | <code>NA</code> plus failure record |
| Not observed after join | No matching source key | <code>NA</code> plus join audit |
| Intentionally withheld | Confidentiality or suppression | Provider code retained separately |

Do not convert these states to zero. Do not remove incomplete rows without
recording the rule and resulting coverage.

Imputation creates estimated values. It requires a method, assumptions, an
indicator of which values were imputed, and—when used for prediction—training-
only estimation.

Missingness summaries should be computed before and after preparation by
important groups such as country, year, source, and variable.

---

## Duplicates and repeated observations

A **duplicate** is not defined merely by two identical-looking rows. It depends
on the expected grain and candidate key.

Repeated records may be legitimate when they differ by:

- item or element;
- measurement method;
- source status or revision;
- unit;
- spatial support;
- time period; or
- quality flag.

When a candidate key is duplicated:

1. preserve the records;
2. inspect variables omitted from the key;
3. consult source documentation;
4. determine whether the proposed grain is wrong;
5. justify any aggregation or correction; and
6. record what information is lost.

Calling <code>distinct()</code> without understanding the key can silently hide
a source or modeling problem.

---

## Outliers and errors

An **outlier** is unusual relative to a distribution or model. An **error** is a
value known to be incorrect under justified evidence. These are not synonyms.

Investigate an unusual value using:

- provider flags and notes;
- units and definitions;
- neighboring periods and related measures;
- source revisions;
- domain knowledge; and
- independent evidence where appropriate.

Possible decisions include:

- retain and report;
- correct through a documented rule;
- exclude for a defined analysis while preserving the source;
- analyze with robust or transformed methods; or
- mark unresolved.

Deleting a point because it changes a result is not a defensible preparation
rule.

---

## Derived variables and transformations

A derived variable should have:

- a precise name and definition;
- input variables and source artifacts;
- a formula or algorithm;
- a unit;
- a valid domain;
- missingness behavior; and
- an interpretation and limitation.

The example project derives:

~~~text
log_yield = log(yield_tonnes_per_hectare)
~~~

The logarithm is defined only for positive yield. A safe transformation returns
missing for zero, negative, or missing inputs and records that rule.

Log transformation changes interpretation: a one-unit change on the log scale
is not a one-tonne-per-hectare change. Retain the untransformed yield so plots,
checks, and interpretations can return to the original scale.

Other common derived variables include:

- rates and proportions;
- differences and percentage changes;
- rolling or grouped summaries;
- categorical bins; and
- interaction terms.

Each introduces assumptions and should be created as late as necessary for its
purpose.

---

## Data types and valid domains

A storage type does not fully specify a variable:

| Variable | Storage type | Domain |
| --- | --- | --- |
| Project country ID | Character | Nine reviewed codes |
| Year | Integer | 1990–2022 |
| Yield | Double | Non-negative, t/ha |
| Season start | Date | 1 October of preceding year |
| Season end | Date | 30 April of ending year |
| Days observed | Integer | 212 or 213 |

Validate domains after parsing and transformation. A numeric value can still
have an invalid unit or range. A character variable can still contain an
unknown code.

Dates should use explicit formats and time semantics. The string "2022" can
mean a calendar year, reporting year, or season-ending year; its type alone
does not resolve that meaning.

---

## Preparation audits

A preparation audit compares inputs, decisions, and outputs.

Useful checks include:

| Check | Example expectation |
| --- | --- |
| Input identity | Expected path or checksum |
| Required columns | All transformation inputs are present |
| Input key | No duplicate area-year-element-unit keys |
| Mappings | Every element-unit combination is recognized |
| Row count | Output equals countries multiplied by years |
| Output key | Country-year is unique |
| Coverage | Nine countries and years 1990–2022 |
| Unit conversion | Converted values reproduce source values |
| Missingness | Changes are explained by documented operations |
| Domain | Log yield missing exactly when yield is not positive |
| Input immutability | Managed input checksum remains unchanged |

Classify checks consistently:

- **pass**: expectation is satisfied;
- **warning**: review is required but the evidence remains usable;
- **failure**: a critical contract is violated and output should not be written;
- **unknown**: code alone cannot decide.

An audit should be machine-readable and summarized for people. It should not
rewrite the input to make its own checks pass.

---

## Lineage and documentation

The prepared dataset needs several complementary records.

### Data dictionary

Define every output variable, type, unit, role, allowed values, missingness,
source, and notes.

### Human-readable documentation

Explain:

- what the dataset represents;
- why it exists;
- how it was prepared;
- intended uses;
- important assumptions;
- limitations; and
- related source and provenance records.

### Provenance and lineage

Record:

- input artifacts;
- preparation script;
- relevant parameters;
- transformations and formulas;
- output artifact;
- creation or regeneration process;
- information loss; and
- relationship to later artifacts.

A compact lineage table might be:

| Output | Input | Operation |
| --- | --- | --- |
| <code>yield_tonnes_per_hectare</code> | FAOSTAT Yield in kg/ha | Divide by 1000 |
| <code>production_tonnes</code> | FAOSTAT Production in t | Reshape |
| <code>harvested_area_hectares</code> | FAOSTAT Area harvested in ha | Reshape |
| <code>log_yield</code> | Prepared yield in t/ha | Natural log for positive values |

The script is executable evidence, but it is not a substitute for these
records.

---

## Preparation and data leakage

A transformation causes **data leakage** when it uses information unavailable
at the time a prediction would be made or lets the test data influence model
training.

Operations that can leak include:

- imputing with the full-dataset mean or median;
- scaling with full-dataset statistics;
- selecting variables using test outcomes;
- deriving categories from the complete distribution;
- choosing outlier thresholds after inspecting test errors;
- using future observations to construct past predictors.

Operations based on fixed external definitions are usually safe before the
split:

- deterministic unit conversion;
- explicit source-label mapping;
- parsing a documented date format;
- reshaping using known source classifications.

When uncertain, separate preparation into:

1. deterministic, source-defined transformations applied consistently; and
2. model-estimated preprocessing fit on the training data and applied unchanged
   to validation or test data.

This boundary becomes central in the Predictive Modeling session.

---

## Idempotence and deterministic outputs

A deterministic preparation produces the same output from the same inputs,
code, parameters, and environment.

An **idempotent** workflow can be rerun safely without accumulating duplicated
rows or repeatedly transforming its own output.

Guidelines:

- read managed inputs, not a previous derived output, unless the dependency is explicit;
- overwrite generated outputs atomically or only after checks pass;
- avoid current-date, random, locale, or row-order dependence unless controlled;
- sort output by explicit keys for stable review;
- use project-relative paths;
- fail visibly on unresolved schemas or mappings; and
- keep network acquisition outside the normal offline preparation run.

Checksums can establish byte identity, but deterministic meaning also depends
on package versions, locale, numeric representation, and output ordering.

---

## Check your understanding

1. How do validation and preparation differ?
2. What information belongs in an analysis contract?
3. Why can long and wide tables both be valid representations?
4. What should you inspect when a wide transformation produces multiple values per cell?
5. Why is missing not equivalent to zero?
6. When is a repeated row not a duplicate?
7. What evidence is needed before calling an outlier an error?
8. Which details should document a derived variable?
9. What should a preparation audit compare?
10. Which preprocessing operations must be fit using training data only?
11. What makes a preparation workflow deterministic and safe to rerun?

---

## Further resources

### Transformation and tidy structure

- [R for Data Science (2e): Data transform](https://r4ds.hadley.nz/data-transform.html)
- [R for Data Science (2e): Data tidying](https://r4ds.hadley.nz/data-tidy.html)
- [dplyr reference](https://dplyr.tidyverse.org/reference/)
- [tidyr reference](https://tidyr.tidyverse.org/reference/)

### Cleaning and reproducibility

- [The Turing Way: Data cleaning](https://book.the-turing-way.org/reproducible-research/rdm/rdm-cleaning/)
- [The Turing Way: Research Data Management](https://book.the-turing-way.org/reproducible-research/rdm/)
- [Data Carpentry: Data Organization in Spreadsheets](https://datacarpentry.org/spreadsheet-ecology-lesson/)

### Modeling preprocessing

- [tidymodels: Preprocess your data with recipes](https://www.tidymodels.org/start/recipes/)
- [Feature Engineering and Selection](https://bookdown.org/max/FES/)

---

## Continue to Application

Continue with [Prepare the maize country-year data](06_data_preparation_application.md). The
application states the preparation contract, inspects the managed FAOSTAT
input, reshapes its elements, converts and derives variables, validates the
country-year panel, integrates CHIRPS, and documents the resulting artifacts.
