# 5.1) Why integrate data?

---

- Last Update: 2026-09-03
- Source: [05_01_data_integration_motivation.md](/learning-modules/intro-ds-module/05_01_data_integration_motivation.md)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Food-systems questions need several sources](#food-systems-questions-need-several-sources)
- [Acquisition supports integration](#acquisition-supports-integration)
- [Integration is more than joining tables](#integration-is-more-than-joining-tables)
- [Common integration dimensions](#common-integration-dimensions)
  - [Identifiers](#identifiers)
  - [Time](#time)
  - [Space](#space)
  - [Units](#units)
  - [Schema and classification](#schema-and-classification)
  - [Grain](#grain)
- [A reproducible integration workflow](#a-reproducible-integration-workflow)
- [What can go wrong](#what-can-go-wrong)
  - [The external source changes](#the-external-source-changes)
  - [The join uses names](#the-join-uses-names)
  - [The key is incomplete](#the-key-is-incomplete)
  - [Unmatched records disappear](#unmatched-records-disappear)
  - [Aggregation changes meaning](#aggregation-changes-meaning)
  - [Credentials leak](#credentials-leak)
- [Across study designs and in the worked example](#across-study-designs-and-in-the-worked-example)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
- [Continue to Concepts](#continue-to-concepts)

---

## Learning objectives

After completing this page, you should be able to:

- explain why a managed dataset may still be insufficient for a question;
- distinguish data management, integration, and preparation;
- identify identifier, temporal, spatial, unit, and schema mismatches;
- describe the main stages of a reproducible integration workflow; and
- explain why every join requires an audit.

---

## Place in the session

This is the **Motivation** part of the Data Integration session:

```text
Motivation  →  Concepts  →  Application
    ↑
 this page
```

The preceding Data Management session established how each project source
should be understood, documented, and validated. This session asks how to
connect managed measurements or datasets without hiding
incompatible identifiers, grains, periods, units, or definitions.

[Understand data-integration concepts](05_02_data_integration_concepts.md) develops the mental
model. The [application](05_03_data_integration_application.md) demonstrates it
with FAOSTAT and CHIRPS and provides transfer questions for other studies.

---

## Food-systems questions need several sources

Changes in maize yield may be related to many processes:

- harvested area and production;
- rainfall and temperature;
- soils and topography;
- fertilizer and irrigation;
- market access and prices;
- policies, conflict, and infrastructure.

No single source contains every relevant concept at the same geographic and temporal resolution.

```text
FAOSTAT -------------------------\
                                  \
Annual climate summaries ----------> country–year dataset
                                  /
Country identifiers -------------/
```

Combining sources enriches an analysis but introduces choices: a result can shift with the provider, query, release, crosswalk, time aggregation, spatial boundary, join type, or unit conversion selected.

---

## Acquisition supports integration

Acquisition belongs to the data lifecycle introduced under Data Management. It
recurs here when CHIRPS is added, in support of the main integration problem:
obtaining an additional source in a form that can be aligned with existing
evidence.

Acquisition determines:

- which provider and dataset are used;
- which variables, countries, years, and quality statuses are requested;
- which version or release enters the analysis;
- which observations are excluded by the query;
- whether the response can be retrieved again;
- whether licensing permits storage and redistribution.

An acquisition script should make these choices visible; a manual download can be documented just as reproducibly if every selection and interaction is recorded.

---

## Integration is more than joining tables

A join combines records that share keys. Scientific integration also asks whether those records refer to compatible concepts.

Two tables may both contain `country` and `year` while differing in:

- geographic definitions;
- calendar versus growing years;
- annual values versus partial-year observations;
- official versus modeled estimates;
- current versus historical borders;
- nominal versus real monetary values;
- tonnes per hectare versus kilograms per hectare.

The software may complete the join without warning. Technical success is not evidence of conceptual compatibility.

---

## Common integration dimensions

### Identifiers

Different providers use different codes and labels. A documented crosswalk is usually safer than joining by names.

### Time

Daily, monthly, seasonal, and annual values require explicit alignment. Aggregating daily rainfall to a calendar year is not necessarily appropriate for a crop growing season.

### Space

Points, administrative polygons, and grid cells represent geography differently. Boundaries, coordinate reference systems, extent, and resolution affect results.

### Units

Values can be compared only when definitions and units are compatible. Unit conversion must be explicit and tested.

### Schema and classification

Column types, categories, quality flags, and commodity classifications may differ or change over time.

### Grain

Both sources must have a known observational grain. Joining a country–year table to a country–year–month table will multiply rows unless the relationship is handled deliberately.

---

## A reproducible integration workflow

```text
Define the question and intended final grain
        ↓
Identify and evaluate an additional source
        ↓
Record metadata, query, version, license, and access date
        ↓
Acquire or select a preserved source snapshot
        ↓
Validate each input schema and key
        ↓
Align identifiers, periods, space, units, and classifications
        ↓
Join or aggregate with explicit expectations
        ↓
Audit matches, row counts, duplicates, and missingness
        ↓
Save the integrated result and its lineage
```

The workflow should stop visibly when a critical expectation fails. A plausible-looking result is dangerous when the pipeline silently discarded or multiplied observations.

---

## What can go wrong

### The external source changes

An endpoint, schema, historical value, classification, or license can change. Record versions, access dates, and checksums, and preserve permitted responses or teaching snapshots.

### The join uses names

`Congo`, `Congo, Rep.`, and `Republic of the Congo` may refer to the same entity. Similar names may also refer to different entities. Use stable codes and a reviewed crosswalk.

### The key is incomplete

A many-to-many join can multiply records unexpectedly. State and test each source key before integration.

### Unmatched records disappear

An inner join can silently remove countries or years. Inspect unmatched keys before choosing how to proceed.

### Aggregation changes meaning

Annual or country-level summaries hide variation and depend on weighting and coverage rules. Document those rules and preserve source quality information.

### Credentials leak

API tokens or passwords embedded in scripts can enter Git history. Store secrets outside the repository and document how users supply them.

---

## Across study designs and in the worked example

Integration may connect an assay table to a sample register, field outcomes to
a randomization file, repeated observations to soil or sensor measurements, or
one external provider dataset to another. In every case, the scientific unit,
identifier, time, space, measurement definition, and relationship between rows
must be explicit before joining.

The maize-yield project supplies one worked example:

The exercise begins with the managed FAOSTAT teaching data and combines:

1. a prepared FAOSTAT panel containing maize production, harvested area, and yield; and
2. CHIRPS precipitation aggregated over country polygons for October–April seasons.

This concrete extension exposes realistic issues:

- provider labels versus stable project identifiers;
- gridded daily estimates versus annual country statistics;
- country polygons and spatial aggregation;
- growing seasons versus reporting years; and
- inherited metadata, provenance, uncertainty, and limitations.

The output is a derived country-year dataset plus a join audit — an input for
association analysis, not evidence that precipitation causes maize-yield
differences.

---

## Check your understanding

1. Why does the managed FAOSTAT dataset not answer every food-systems question?
2. Give an example of a join that is technically successful but scientifically invalid.
3. What should be checked before and after every join?
4. Why is a country-code crosswalk itself a dataset that requires documentation?
5. What is the difference between integration and data preparation?
6. Why can students use fixed snapshots even though acquisition remains reproducible?

---

## Further resources

- [Research Data Management — The Turing Way](https://book.the-turing-way.org/reproducible-research/rdm/) connects acquisition, documentation, storage, sharing, and preservation across the research-data lifecycle.
- [HTTP overview — MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) explains the request–response model used by web APIs.
- [Organising data — UK Data Service](https://ukdataservice.ac.uk/learning-hub/research-data-management/format-your-data/organising/) places meaningful filenames, formats, and folder structures within practical research-data management.
- Wickham, H., Çetinkaya-Rundel, M., and Grolemund, G., [R for Data Science (2e): Joins](https://r4ds.hadley.nz/joins.html), explains mutating joins, filtering joins, keys, and common relationship problems in `dplyr`.
- [World Bank Indicators API documentation](https://datahelpdesk.worldbank.org/knowledgebase/topics/125589-developer-information) provides a realistic public API for practising parameterized acquisition and pagination.

---

## Continue to Concepts

Continue with [Understand data-integration concepts](05_02_data_integration_concepts.md),
which covers evaluating a source, aligning grains and identifiers, stating
join relationships, and auditing and preserving lineage.
