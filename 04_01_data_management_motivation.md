# 4.1) Why manage data?

---

- Source: [04_01_data_management_motivation.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/04_01_data_management_motivation.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/04_01_data_management_motivation.md)
- Feedback: [Topic 04: Data Management](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/5)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [The session learning path](#the-session-learning-path)
- [Reproducible does not automatically mean correct](#reproducible-does-not-automatically-mean-correct)
- [Data are not self-explanatory](#data-are-not-self-explanatory)
- [The data lifecycle](#the-data-lifecycle)
- [What data management contributes](#what-data-management-contributes)
  - [Meaning](#meaning)
  - [Origin and history](#origin-and-history)
  - [Quality](#quality)
  - [Organization](#organization)
  - [Responsibility](#responsibility)
- [Why this matters in food-systems research](#why-this-matters-in-food-systems-research)
- [What data management cannot guarantee](#what-data-management-cannot-guarantee)
- [Across study designs and in the worked example](#across-study-designs-and-in-the-worked-example)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
- [Continue to Concepts](#continue-to-concepts)

---

## Learning objectives

After completing this page, you should be able to:

- explain why reproducible code is not sufficient for trustworthy research;
- describe the stages of a simple data lifecycle;
- distinguish data management from data preparation and analysis;
- identify common risks in food-systems datasets; and
- explain which project artifacts make data understandable and auditable.

---

## The session learning path

The Data Management session follows three connected parts:

```text
Motivation             Concepts                 Application
Why manage data?  →    How do we understand, →  How do we document and
                       describe, and assess      validate the maize data?
                       a dataset?
```

This page provides the **Motivation**. It establishes the problem and the purpose of data management. The [Concepts page](04_02_data_management_concepts.md) develops the vocabulary and mental models. The [Application page](04_04_data_management_application.md) demonstrates them with a supplied secondary dataset and explains how to transfer the workflow to other study designs.

The goal is to explain why each project artifact is needed and which risk it addresses, not merely to complete a sequence of commands.

---

## Reproducible does not automatically mean correct

Imagine that a laboratory, field, or secondary-data workflow is completely reproducible:

- Git records every change to the scripts;
- `renv` restores the required R packages;
- Docker supplies the operating-system environment;
- the same command produces the same report.

The result can still be wrong or misleading if:

- a measurement is interpreted using the wrong unit;
- a blank, instrument code, or provider-specific missing code is read as zero;
- biological samples and technical replicates are confused;
- treatment, plot, visit, site, or entity identifiers are misaligned;
- an estimated value is treated as a direct observation; or
- a protocol, instrument calibration, or external source changes unnoticed.

Reproducibility answers, "Can the workflow be repeated?" Data management also asks, "What evidence entered the workflow, what does it mean, and is it suitable for this use?"

---

## Data are not self-explanatory

A rectangular file may look simple:

```text
Area          Year   Element       Unit    Value   Flag
Zambia        2020   Production    t       ...     A
Zambia        2020   Area harvested ha     ...     E
Zambia        2020   Yield         kg/ha   ...     I
```

The values cannot be interpreted safely without additional information:

- What does each element mean?
- Is `t` tonnes or another unit?
- What do the flags `A`, `E`, and `I` mean?
- Are the values measured, estimated, imputed, or calculated?
- Does `2020` refer to a calendar year, harvest year, or reporting period?
- Which geographic definition of Zambia is used?
- Which variables together identify a unique observation?

Metadata, code lists, methodological notes, and provenance are therefore part of the data evidence—not optional decoration.

---

## The data lifecycle

Data move through connected stages:

```text
Plan
  ↓
Acquire
  ↓
Describe and organize
  ↓
Validate
  ↓
Prepare and analyze
  ↓
Share
  ↓
Preserve or dispose
```

The lifecycle is not strictly linear: validation may reveal that the wrong source was acquired, analysis may expose an incomplete data dictionary, or a new release may require repeating acquisition and checks.

Decisions made early constrain later work. An unrecorded source, unit, license, or grain can leave later analysts unable to interpret or share the results responsibly.

---

## What data management contributes

Good data management makes several aspects of a project explicit.

### Meaning

- the observational unit and dataset grain;
- variable definitions, units, classifications, and flags;
- the difference between identifiers, labels, and measurements.

### Origin and history

- provider and dataset;
- version or release;
- access date and retrieval method;
- transformations that produced derived artifacts.

### Quality

- expected structure and coverage;
- validation rules and results;
- known anomalies, limitations, and unresolved questions.

### Organization

- which files are raw inputs;
- which files are derived;
- which scripts create each output;
- which data should or should not be stored in Git.

### Responsibility

- license and citation requirements;
- whether redistribution is permitted;
- privacy, confidentiality, security, and access constraints;
- how long data should be retained.

---

## Why this matters in food-systems research

Food-systems analyses often combine observations from different institutions and purposes. Common challenges include:

- changing country or administrative boundaries;
- different commodity and land-use classifications;
- annual, seasonal, monthly, and daily reporting periods;
- estimated, imputed, modeled, and directly observed values;
- measurements at farm, household, market, district, country, or grid-cell level;
- multiple unit conventions;
- incomplete coverage of informal activities or marginalized populations;
- revisions to historical series.

A dataset can be internally tidy and technically valid while still being unsuitable for a particular research question. Fitness for purpose requires scientific judgment and knowledge of how the values were produced.

---

## What data management cannot guarantee

Data management can improve transparency and reduce avoidable errors. It cannot guarantee:

- that the provider measured the intended concept accurately;
- that the sample represents the target population;
- that missing groups or activities are unimportant;
- that a documented assumption is scientifically valid;
- that a FAIR dataset is open or unrestricted;
- that a reproducible result supports a causal conclusion.

Documentation makes limitations visible. It does not remove them.

---

## Across study designs and in the worked example

The source of data changes what must be documented, but not the need for data
management. Laboratory projects emphasize sample lineage, assays, batches, and
calibration. Field experiments emphasize randomization, treatments, blocks,
plots, and protocol deviations. Field observational studies emphasize sampling,
repeated visits, measurement context, consent, and selection. Secondary-data
projects emphasize provider definitions, versions, access, revisions, and
licenses.

The module then demonstrates these shared principles with the maize-yield
project.

The Data Management session produces a documented package around a supplied FAOSTAT extract:

```text
maize-yield-project/
├── data/
│   └── input/
│       └── faostat-maize-yield-sample.csv
├── metadata/
│   ├── faostat-maize-yield-data-dictionary.csv
│   └── provenance.yml
├── scripts/
│   └── validate-data.R
└── reports/
    └── data-validation.html
```

The goal is not yet to clean or model the data. The goal is to make it possible for another person to determine:

- what one row represents;
- what each variable, unit, code, and flag means;
- where the data came from;
- which validation checks passed or failed;
- which concerns remain; and
- how the data may be used and shared.

---

## Check your understanding

1. A report can be reproduced exactly. Name three reasons why its results could still be misleading.
2. Why are provider metadata part of the evidence rather than optional background reading?
3. Which information belongs in a provenance record?
4. Why does "raw" describe a role rather than a guarantee of quality?
5. Give one example of a technically valid food-systems dataset that may not be fit for a particular research question.
6. Which questions should be answered before data are committed to Git or shared publicly?

---

## Further resources

Use these resources to deepen the motivation and place this session in the wider research-data-management landscape:

- [Research Data Management — The Turing Way](https://book.the-turing-way.org/reproducible-research/rdm/) connects storage, documentation, metadata, sharing, and FAIR practice to reproducible research.
- [Overview of Reproducible Research — The Turing Way](https://book.the-turing-way.org/reproducible-research/overview.html) places data planning, processing, and reuse within the wider research lifecycle.
- [Research Data Management — UK Data Service](https://ukdataservice.ac.uk/learning-hub/research-data-management/) is a practical learning hub covering the data lifecycle, rights, storage, and sharing.
- [Data management checklist — UK Data Service](https://ukdataservice.ac.uk/learning-hub/research-data-management/plan-to-share/checklist/) offers planning questions on documentation, access, and preservation.
- Wilkinson, M. D. et al. (2016), [The FAIR Guiding Principles for scientific data management and stewardship](https://doi.org/10.1038/sdata.2016.18), introduces the Findable, Accessible, Interoperable, and Reusable principles.

---

## Continue to Concepts

Continue with [Understand data-management concepts](04_02_data_management_concepts.md). It introduces dataset grain, keys, structures, metadata, provenance, quality, project organization, and responsible use—the concepts required by the application.
