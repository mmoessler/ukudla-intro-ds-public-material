# Why prepare data?

---

- Last Update: 2026-08-21
- Source: [06_data_preparation_motivation.md](/learning-modules/intro-ds-module/06_data_preparation_motivation.md)

---

## Outline

- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Managed and integrated data are not automatically analysis-ready](#managed-and-integrated-data-are-not-automatically-analysis-ready)
- [Preparation decisions are analytical decisions](#preparation-decisions-are-analytical-decisions)
- [Preparation happens around integration](#preparation-happens-around-integration)
- [Preparation should create new artifacts](#preparation-should-create-new-artifacts)
- [Common preparation decisions](#common-preparation-decisions)
- [What can go wrong](#what-can-go-wrong)
- [How this connects to the maize-yield project](#how-this-connects-to-the-maize-yield-project)
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

The preceding Data Management session made the source artifacts understandable
and auditable. Data Integration connected the FAOSTAT maize statistics with
CHIRPS growing-season precipitation. This session asks how those managed and
integrated data should be represented for a defined analysis.

[Understand data-preparation concepts](06_data_preparation_concepts.md) explains the relevant
distinctions and transformation patterns. [Prepare the maize country-year
data](06_data_preparation_application.md) applies them in the example project.

---

## Managed and integrated data are not automatically analysis-ready

A dataset can be well documented, validated, and successfully integrated while
still having an inconvenient or inappropriate structure for analysis.

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

Neither shape is universally superior. The appropriate representation depends
on what one analytical observation should mean and which comparisons or models
will be performed.

---

## Preparation decisions are analytical decisions

Data preparation is sometimes described as routine housekeeping. In practice,
its choices affect every later result.

Examples include:

- filtering changes which observations and population are represented;
- aggregation changes the grain and hides variation;
- reshaping changes what is represented by rows and columns;
- unit conversion changes numeric values while aiming to preserve meaning;
- recoding can merge or separate categories;
- missing-value handling changes which values enter a calculation;
- outlier treatment changes the distribution;
- derived variables encode formulas and assumptions.

A script can execute these operations correctly while implementing an
inappropriate scientific decision. Preparation therefore begins with an
**analysis contract** that states the intended population, grain, key,
variables, units, periods, and missing-value policy.

---

## Preparation happens around integration

The teaching sequence presents Data Integration before Data Preparation, but a
real workflow is not a single linear sequence of topic labels.

Source-specific preparation may be required before two sources can be joined:

~~~text
FAOSTAT long input
        ↓
reshape to a country-year maize panel
        ↓
integrate with country-season CHIRPS data
~~~

Further preparation may occur after integration:

~~~text
integrated country-year data
        ↓
select analysis variables
        ↓
derive transformations or features
        ↓
visualize, describe, or model
~~~

This does not weaken the distinction between topics. It clarifies their
different teaching focus:

| Activity | Main question |
| --- | --- |
| Data management | What do the data mean, and how are they governed? |
| Data integration | How are compatible observations connected across sources? |
| Data preparation | How are data represented for a defined analysis? |

Workflow order follows dependencies. Documentation should make each decision
visible even when several activities occur in one script.

---

## Preparation should create new artifacts

Managed input data should not be edited in place.

A reproducible pattern is:

~~~text
managed input
    + preparation script
    + documented parameters
        ↓
derived output
    + checks
    + dictionary
    + lineage
~~~

This pattern offers several benefits:

- the original evidence remains available for comparison;
- every transformation can be reviewed in version control;
- the output can be regenerated;
- alternative preparations can coexist;
- errors can be traced to a specific step; and
- downstream users can determine how a value was created.

A derived dataset is not less important than a source dataset. It requires its
own grain, key, variable definitions, limitations, and provenance.

---

## Common preparation decisions

### Select and filter

Retain only variables and observations required by the analysis, but state the
inclusion rule and report how coverage changes.

### Parse and recode

Convert text to numbers or dates and map labels to categories. Failed parsing
or unmapped values should be reported rather than silently turned into missing
values.

### Reshape

Change between long and wide forms only after the input grain and candidate key
are known. Unexpected duplicate cells usually indicate an incomplete key or an
unresolved aggregation decision.

### Convert units

Record the source unit, target unit, formula, and valid domain. A unit label
should never be changed without changing the corresponding values.

### Address missingness

Distinguish unavailable measurements, structural absence, invalid values, and
missingness introduced by a join or transformation. Do not replace missing
values merely because a later function rejects them.

### Derive variables

Document formulas, inputs, units, valid domains, and interpretation. Retain the
untransformed variable when it is useful for review.

---

## What can go wrong

### The source is edited manually

A spreadsheet correction destroys the boundary between source evidence and
derived decisions. Preserve the input and express justified corrections in
code.

### Filtering silently changes the population

Removing incomplete countries or years can create a more convenient table
while changing the research question. Report inclusion criteria and compare
coverage before and after.

### Reshaping multiplies or loses observations

A wide transformation requires at most one value for each target cell. Test the
input key rather than choosing an aggregation function only to suppress a
warning.

### Missing values become zero

Zero is an observed value with meaning. Missing is an absence of a usable
value. They are not interchangeable.

### Outliers are deleted because they look unusual

An extreme value may be an error, a legitimate event, or an important part of
the phenomenon. Investigate it using definitions, flags, and source evidence.

### A transformation leaks test information

Imputation values, scaling parameters, selected features, or category handling
learned from the complete dataset can reveal the test period to a predictive
model. Such operations must later be estimated using training data only.

### The output has no documentation

A clear script does not replace a dictionary or readable dataset description.
Users need to understand the output without reconstructing every expression.

---

## How this connects to the maize-yield project

The example project currently performs source-specific preparation in
<code>scripts/prepare-maize-data.R</code>. It:

1. reads the fixed FAOSTAT teaching input;
2. checks required fields, element-unit combinations, and candidate-key uniqueness;
3. retains maize observations;
4. maps provider elements to project measure names;
5. reshapes the long values into a country-year panel;
6. converts yield from kilograms per hectare to tonnes per hectare;
7. derives log yield for positive values; and
8. writes <code>data/derived/maize-yield-panel.csv</code>.

The integration script then maps project identifiers and adds CHIRPS
precipitation. The resulting
<code>data/derived/maize-yield-with-precipitation.csv</code> is the starting
point for visualization, descriptive analysis, and modeling.

The preparation session makes the contracts and evidence behind those
transformations explicit. It also identifies improvements that can later be
implemented in the example repository, such as a dedicated preparation audit
and human-readable documentation for the intermediate maize panel.

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
- [R for Data Science (2e): Data tidying](https://r4ds.hadley.nz/data-tidy.html)
  explains observations, variables, values, and reshaping with
  <code>tidyr</code>.
- [The Turing Way: Data cleaning](https://book.the-turing-way.org/reproducible-research/rdm/rdm-cleaning/)
  discusses reproducible cleaning, preservation of raw data, and documented
  decisions.
- [Data Carpentry: Data Organization in Spreadsheets](https://datacarpentry.org/spreadsheet-ecology-lesson/)
  explains common structural problems and good tabular-data practices.
- [tidymodels: Data preprocessing](https://www.tidymodels.org/start/recipes/)
  introduces preprocessing estimated within a modeling workflow and is useful
  for understanding leakage.

---

## Continue to Concepts

Continue with [Understand data-preparation concepts](06_data_preparation_concepts.md). It
develops the analysis contract, tidy-data model, transformation vocabulary,
missingness and anomaly decisions, preparation audits, lineage, and leakage
boundary.
