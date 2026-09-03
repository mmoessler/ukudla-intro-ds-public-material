# 6.1) Why prepare data?

---

- Last Update: 2026-09-03
- Source: [06_01_data_preparation_motivation.md](/learning-modules/intro-ds-module/06_01_data_preparation_motivation.md)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Managed and integrated data are not automatically analysis-ready](#managed-and-integrated-data-are-not-automatically-analysis-ready)
- [Preparation decisions are analytical decisions](#preparation-decisions-are-analytical-decisions)
- [Preparation happens around integration](#preparation-happens-around-integration)
- [Preparation should create new artifacts](#preparation-should-create-new-artifacts)
- [Common preparation decisions](#common-preparation-decisions)
- [What can go wrong](#what-can-go-wrong)
- [Across study designs and in the worked example](#across-study-designs-and-in-the-worked-example)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
- [Continue to Concepts](#continue-to-concepts)

---

## Learning objectives

After completing this page, you should be able to:

- explain why managed and integrated data may still be unsuitable for analysis;
- describe how preparation choices affect the population, variables, and meaning represented;
- distinguish validation, cleaning, integration, and preparation;
- explain why preparation may occur before and after integration;
- identify common risks of undocumented or destructive transformations; and
- describe the role of an analysis contract, audit, dictionary, and lineage record.

---

## Place in the session

This is the **Motivation** part of the Data Preparation session:

~~~text
Motivation  →  Concepts  →  Application
    ↑
 this page
~~~

The preceding sessions made source artifacts understandable and auditable and
showed how compatible observations can be integrated. This session asks how
managed evidence should be represented for a defined analysis.

[Understand data-preparation concepts](06_02_data_preparation_concepts.md) explains the relevant
distinctions and transformation patterns. The [application](06_03_data_preparation_application.md)
demonstrates them with the maize project and maps them to other study designs.

---

## Managed and integrated data are not automatically analysis-ready

A dataset can be well documented, validated, and integrated while still having
an inconvenient structure for analysis.

The fixed FAOSTAT input uses a long representation:

| area | year | element | unit | value |
| --- | ---: | --- | --- | ---: |
| Zambia | 2022 | Yield | kg/ha | 2340 |
| Zambia | 2022 | Production | t | 3621000 |
| Zambia | 2022 | Area harvested | ha | 1547000 |

This representation preserves the provider's element and unit fields. A
country-year analysis generally needs a separate column for each retained
measure:

| country | year | yield_tonnes_per_hectare | production_tonnes | harvested_area_hectares |
| --- | ---: | ---: | ---: | ---: |
| Zambia | 2022 | 2.34 | 3621000 | 1547000 |

Neither shape is universally superior; the right representation depends on
what one observation should mean and which comparisons follow.

---

## Preparation decisions are analytical decisions

Data preparation is sometimes treated as routine housekeeping, but its choices
affect every later result: filtering changes the represented population,
aggregation changes the grain, reshaping changes what rows and columns mean,
unit conversion changes numeric values, recoding can merge or separate
categories, and missing-value or outlier treatment changes the distribution.

A script can execute these operations correctly while implementing an
inappropriate scientific decision. Preparation therefore begins with an
**analysis contract** that states the intended population, grain, key,
variables, units, periods, and missing-value policy.

---

## Preparation happens around integration

The teaching sequence presents Data Integration before Data Preparation, but a
real workflow is not a single linear pass through topic labels. Source-specific
preparation (reshaping FAOSTAT into a country-year panel) is often needed
before two sources can be joined, and further preparation (selecting variables,
deriving transformations) can follow integration:

~~~text
FAOSTAT long input → reshape → integrate with CHIRPS → select/derive → analyze
~~~

This does not weaken the distinction between topics; it clarifies their
different teaching focus:

| Activity | Main question |
| --- | --- |
| Data management | What do the data mean, and how are they governed? |
| Data integration | How are compatible observations connected across sources? |
| Data preparation | How are data represented for a defined analysis? |

Workflow order follows dependencies; documentation should make each decision
visible even when several activities occur in one script.

---

## Preparation should create new artifacts

Managed input data should not be edited in place. A reproducible pattern
applies a documented script to the managed input and writes a derived output
with its own checks, dictionary, and lineage. This keeps the original evidence
available for comparison, lets every transformation be reviewed in version
control, allows the output to be regenerated, and lets errors be traced to a
specific step.

A derived dataset is not less important than a source dataset. It requires its
own grain, key, variable definitions, limitations, and provenance.

---

## Common preparation decisions

- **Select and filter** — retain only the variables and observations the
  analysis requires, but state the inclusion rule and report how coverage
  changes.
- **Parse and recode** — convert text to numbers, dates, or categories; report
  failed parses and unmapped values rather than silently turning them missing.
- **Reshape** — change between long and wide forms only once the input grain
  and candidate key are known; unexpected duplicate cells usually mean an
  incomplete key.
- **Convert units** — record the source unit, target unit, formula, and valid
  domain; never change a unit label without changing the values.
- **Address missingness** — distinguish unavailable measurements, structural
  absence, invalid values, and join-induced missingness; do not replace a
  missing value merely because a later function rejects it.
- **Derive variables** — document formulas, inputs, units, valid domains, and
  interpretation, and retain the untransformed variable for review.

---

## What can go wrong

- **The source is edited manually** — a spreadsheet correction destroys the
  boundary between source evidence and derived decisions; preserve the input
  and express corrections in code.
- **Filtering silently changes the population** — removing incomplete
  countries or years can change the research question; report inclusion
  criteria and coverage before and after.
- **Reshaping multiplies or loses observations** — a wide transformation
  requires at most one value per target cell; test the input key rather than
  choosing an aggregation function to suppress a warning.
- **Missing values become zero** — zero is an observed value with meaning;
  missing is the absence of a usable value; they are not interchangeable.
- **Outliers are deleted because they look unusual** — an extreme value may be
  an error, a legitimate event, or part of the phenomenon; investigate it
  using definitions, flags, and source evidence.
- **A transformation leaks test information** — imputation values, scaling
  parameters, or selected features learned from the complete dataset can
  reveal the test period to a predictive model; such operations must be
  estimated from training data only.
- **The output has no documentation** — a clear script does not replace a
  dictionary or dataset description; users need to understand the output
  without reconstructing every expression.

---

## Across study designs and in the worked example

Preparation may organize laboratory replicates and detection-limit flags,
preserve treatment and block variables in an experiment, align repeated visits
in an observational study, or reshape provider records. The intended analysis
determines whether a row represents a sample, experimental unit, unit-occasion,
event, or entity-period and which design information must remain available.

The maize-yield project demonstrates one repeated entity-period workflow.

The example project performs source-specific preparation in
<code>scripts/prepare-maize-data.R</code>: it validates the fixed FAOSTAT
input, maps provider elements to project measure names, reshapes the long
values into a country-year panel, converts yield to tonnes per hectare,
derives log yield for positive values, and writes
<code>data/derived/maize-yield-panel.csv</code>. The integration script then
maps project identifiers and adds CHIRPS precipitation, producing
<code>data/derived/maize-yield-with-precipitation.csv</code> — the starting
point for visualization, descriptive analysis, and modeling.

This session makes the contracts and evidence behind those transformations
explicit, and identifies future improvements such as a dedicated preparation
audit and documentation for the intermediate maize panel.

---

## Check your understanding

1. Why can a validated dataset still require preparation?
2. Give an example of a preparation decision that changes the represented population.
3. Why must the input key be known before reshaping from long to wide?
4. What is the difference between a missing value and zero?
5. Why should an unusual value not be removed automatically?
6. Why can preparation occur both before and after integration?
7. Which preparation operations can cause leakage in predictive modeling?
8. What evidence should accompany an analysis-ready dataset?

---

## Further resources

- Wickham, H., Çetinkaya-Rundel, M., and Grolemund, G.,
  [R for Data Science (2e): Data transform](https://r4ds.hadley.nz/data-transform.html)
  introduces filtering, mutation, grouping, and summarization with
  <code>dplyr</code>.
- [The Turing Way: Data cleaning](https://book.the-turing-way.org/reproducible-research/rdm/rdm-cleaning/)
  discusses reproducible cleaning, preservation of raw data, and documented
  decisions.
- [tidymodels: Data preprocessing](https://www.tidymodels.org/start/recipes/)
  introduces preprocessing estimated within a modeling workflow and is useful
  for understanding leakage.

---

## Continue to Concepts

Continue with [Understand data-preparation concepts](06_02_data_preparation_concepts.md). It
develops the analysis contract, tidy-data model, transformation vocabulary,
missingness and anomaly decisions, preparation audits, lineage, and leakage
boundary.
