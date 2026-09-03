# 4.2) Understand data-management concepts

---

- Last Update: 2026-09-03
- Source: [04_02_data_management_concepts.md](/learning-modules/intro-ds-module/04_02_data_management_concepts.md)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Observations, variables, and values](#observations-variables-and-values)
- [Dataset grain](#dataset-grain)
- [Identifiers, labels, and keys](#identifiers-labels-and-keys)
- [Common data structures](#common-data-structures)
  - [Cross-sectional data](#cross-sectional-data)
  - [Time-series data](#time-series-data)
  - [Panel data](#panel-data)
  - [Hierarchical data](#hierarchical-data)
  - [Spatial vector data](#spatial-vector-data)
  - [Raster or gridded data](#raster-or-gridded-data)
- [Metadata](#metadata)
- [Data dictionaries](#data-dictionaries)
- [Provenance](#provenance)
- [Data quality and fitness for purpose](#data-quality-and-fitness-for-purpose)
- [Project organization](#project-organization)
- [Licensing, privacy, and FAIR principles](#licensing-privacy-and-fair-principles)
- [From concepts to application](#from-concepts-to-application)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
  - [Research data and the lifecycle](#research-data-and-the-lifecycle)
  - [Data structure and organization](#data-structure-and-organization)
  - [Metadata and documentation](#metadata-and-documentation)
  - [FAIR principles](#fair-principles)

---

## Learning objectives

After completing this page, you should be able to:

- state the observational unit and grain of a dataset;
- distinguish identifiers, labels, measures, units, and flags;
- propose and test a candidate key;
- recognize common tabular, temporal, hierarchical, and spatial structures;
- distinguish source metadata, a project data dictionary, and provenance;
- organize data artifacts by their role in a workflow; and
- identify basic conditions for responsible storage and reuse.

---

## Place in the session

This is the **Concepts** part of the Data Management session:

```text
Motivation  →  Concepts  →  Application
                ↑
             this page
```

The [Motivation page](04_01_data_management_motivation.md) explains why reproducible computation is not sufficient when data are misunderstood. This page provides reusable vocabulary and decisions demonstrated in the [maize-yield worked example](04_04_data_management_application.md).

Use the concepts as questions to ask of a dataset, not only as definitions to memorize:

- What does one row represent?
- Which variables identify it?
- Which metadata give its values meaning?
- How can its structure and meaning be checked?
- Under which conditions may it be stored, used, or shared?

---

## Observations, variables, and values

For a rectangular dataset:

- an **observation** records information about one unit in one defined context;
- a **variable** records one property across observations;
- a **value** is the recorded content for one variable in one observation.

These definitions depend on the intended structure. An observation might be a
sample-assay, a plot-treatment-occasion, a farm visit, or an entity-period
record. Technical replicates and repeated measurements are not automatically
independent observations; their role follows from the study design.

Variables can play different roles:

| Role | Example | Purpose |
| --- | --- | --- |
| Identifier | Sample, plot, farm, or entity ID | Identifies a unit |
| Label | Site or treatment label | Displays a readable name |
| Time | Collection date, visit, season, or year | Locates an observation in time |
| Classification | Assay, treatment, variable, or item code | Identifies a defined category |
| Measure | Observed value | Records a quantity |
| Unit | Measurement unit | Defines the measurement scale |
| Quality information | Laboratory, field, or provider flag | Qualifies how a value was obtained |

Do not assume that a numeric column is a measure. Codes may be stored as numbers but should not be added or averaged.

---

## Dataset grain

The **grain** states what one row represents. Write it as a sentence:

> One row represents one element for one item in one area and year, expressed in one unit.

Grain is important because it determines:

- which variables should identify a row uniquely;
- which comparisons are meaningful;
- whether a join will preserve or multiply observations;
- whether an aggregation changes the scientific meaning.

Before using a dataset, answer:

1. What real-world or statistical unit does one row represent?
2. At what temporal resolution is it observed?
3. At what geographic resolution is it observed?
4. Which classifications distinguish otherwise similar rows?
5. Can more than one record exist for the same apparent unit?

---

## Identifiers, labels, and keys

An **identifier** is intended to distinguish an entity. A **label** is intended for people to read.

Identifiers are usually safer for matching because labels can differ in spelling, punctuation, language, abbreviation, capitalization, or naming convention over time.

A **candidate key** is a set of variables expected to identify each row uniquely. A **composite key** uses more than one variable.

Candidate keys reflect the study grain. Examples include:

```text
sample_id + assay + replicate
plot_id + treatment + measurement_date
entity_id + period + variable_code + unit
```

The exact key must be inferred from documentation and tested against the data. Finding duplicates does not automatically mean that one row should be deleted. The proposed key may be incomplete, or the source may intentionally publish multiple statuses, methods, or revisions.

---

## Common data structures

### Cross-sectional data

Many units observed in one context, such as laboratory samples in one batch or
field plots at harvest.

### Time-series data

One unit observed through ordered occasions, such as repeated sensor readings
at one site.

### Panel data

Many units observed repeatedly, such as plots across visits or regions across years.

### Hierarchical data

Units are nested, such as aliquots within samples and batches, plots within
blocks and sites, or farms within districts. Records within the same group may
not be independent.

### Spatial vector data

Locations are represented by points, lines, or polygons. Examples include markets, roads, and administrative boundaries.

### Raster or gridded data

Space is divided into cells. Climate and satellite products often use rasters. Resolution, extent, and coordinate reference system are essential metadata.

These structures can coexist. A panel can be spatial, and sensor observations can form several time series at point locations.

---

## Metadata

Metadata are data about data. Useful source metadata may describe concepts and definitions, collection or estimation methods, population and geographic coverage, temporal coverage and frequency, classifications and code lists, units and conversion rules, missing-value conventions, quality flags and revisions, and license and citation requirements.

Metadata should be read before interpreting values. A familiar column name does not guarantee a familiar definition.

Keep a provider's metadata or a stable reference to it when permitted. External pages and code lists may change.

---

## Data dictionaries

A project data dictionary documents the variables actually used by the project. It complements rather than replaces provider metadata.

A suggested structure is:

```csv
variable,label,definition,type,unit,role,allowed_values,missing_values,source
Area_Code,Area code,FAOSTAT area identifier,character,,identifier,...,,FAOSTAT
Year,Year,Reporting year,integer,year,time,1990-2022,,FAOSTAT
Value,Value,Recorded value,double,varies,measure,,,FAOSTAT
```

Recommended fields:

- source variable name;
- project variable name, if different;
- label and precise definition;
- data type;
- unit;
- role in the dataset;
- allowed values or code-list reference;
- missing-value representation;
- source and methodological note.

Avoid definitions that merely repeat the variable name. "Yield: yield value" does not explain the concept, calculation, or unit.

---

## Provenance

Provenance records where data came from and what happened to them.

A small project can use YAML:

```yaml
artifact: data/input/faostat-maize-yield-sample.csv
provider: Food and Agriculture Organization of the United Nations
dataset: FAOSTAT crop and livestock products
release: "record the release if supplied"
accessed: YYYY-MM-DD
source: "record the URL, endpoint, or delivery method"
license: "record the provider's license or terms"
retrieval: "manual snapshot supplied for the exercise"
checksum_sha256: "record the checksum"
```

For a derived file, also record:

- input artifacts;
- script or workflow step;
- relevant parameters;
- aggregation or conversion rules;
- output artifact;
- known information loss.

Provenance should allow a person to trace a result back to its sources. It is not the same as a data dictionary, which explains variables.

---

## Data quality and fitness for purpose

Data quality is not one universal score. It describes whether data are suitable for an intended use and whether their relevant limitations are understood.

Useful dimensions include:

| Dimension | Question |
| --- | --- |
| Completeness | Are the expected variables, entities, periods, and values present? |
| Validity | Do values follow documented types, formats, codes, and rules? |
| Uniqueness | Does each expected key identify no more than one observation? |
| Consistency | Do related values, units, classifications, and records agree? |
| Plausibility | Are values credible given the concept and context? |
| Timeliness | Are the reference period and release current enough for the intended use? |
| Accuracy | How closely do values represent the real quantity or state of interest? |

Accuracy often cannot be established from the file alone. It may require knowledge of collection methods, sampling, measurement error, estimation, revision, and comparison with independent evidence.

Validation converts justified expectations into repeatable checks. A failed check does not always prove that the source is wrong: the expectation may be wrong, the key may be incomplete, or the data may document a legitimate exception. Preserve the evidence and investigate before correcting or excluding records.

---

## Project organization

A useful starting structure is:

```text
maize-yield-project/
├── data/
│   ├── source/        # complete acquired source artifacts
│   ├── input/         # fixed, managed teaching or analysis inputs
│   └── derived/       # outputs created reproducibly from inputs
├── metadata/          # dictionaries, code lists, licenses, provenance
├── scripts/           # acquisition, validation, integration, preparation
└── reports/           # validation, analysis, and communication
```

Guidelines:

- Do not manually edit raw source files.
- Use code to create derived files.
- Document the connection between every output and its inputs.
- Keep credentials outside scripts and version control.
- Do not commit data merely because they fit in Git.
- Use `.gitignore` for files that are large, sensitive, restricted, or reproducibly retrieved.
- Preserve a source snapshot when permitted and necessary for reproducibility.
- Use checksums or source releases to identify exact inputs.

---

## Licensing, privacy, and FAIR principles

Before storing, processing, or sharing data, determine:

- who owns or controls the data;
- which license or terms apply;
- whether attribution is required;
- whether raw or derived data may be redistributed;
- whether the data contain personal, confidential, commercially sensitive, or location-sensitive information;
- who should have access and how access is controlled;
- when data should be archived or deleted.

FAIR data are:

- **Findable**;
- **Accessible** under clearly stated conditions;
- **Interoperable** through shared formats, vocabularies, and identifiers;
- **Reusable** because meaning, provenance, and conditions are documented.

FAIR does not mean that every person must have unrestricted access. Sensitive data can be FAIR when their existence, metadata, access process, and conditions are clear.

---

## From concepts to application

The application turns each concept into an inspectable project artifact or check:

| Concept | Application evidence |
| --- | --- |
| Grain and key | One-sentence grain statement and duplicate-key check |
| Metadata | Variable definitions, units, code lists, and flags |
| Data dictionary | `metadata/data-dictionary.csv` |
| Provenance | `metadata/provenance.yml` and source checksum |
| Data quality | Structural and semantic validation checks |
| Project organization | Raw input remains unchanged; derived artifacts have separate roles |
| Responsible use | License, citation, access, and sharing conditions are recorded |

Continue with [Manage the maize-yield data](04_04_data_management_application.md). During the exercise, return to this page whenever a task requires a definition or scientific decision.

---

## Check your understanding

1. Write the grain of a country–year–item–element dataset in one sentence.
2. Why should country codes usually be used instead of country names for matching?
3. How would you investigate duplicate candidate keys before deciding to remove anything?
4. What is the difference between provider metadata, a data dictionary, and provenance?
5. Why might a raw source file be excluded from Git?
6. Can restricted data satisfy FAIR principles? Explain your answer.

---

## Further resources

The following resources expand particular concepts from this page:

### Research data and the lifecycle

- [Research Data — The Turing Way](https://book.the-turing-way.org/reproducible-research/rdm/rdm-data/) introduces research data, the data lifecycle, and data-management planning.
- [Research Data Management — The Turing Way](https://book.the-turing-way.org/reproducible-research/rdm/) links documentation, storage, sharing, preservation, and FAIR practice to reproducibility.

### Data structure and organization

- [Data Organisation in Spreadsheets — The Turing Way](https://book.the-turing-way.org/reproducible-research/rdm/rdm-spreadsheets.html) explains observations, variables, values, consistent representation, and validation for spreadsheet data.
- [Data Carpentry lessons](https://datacarpentry.org/lessons/) provide beginner-friendly, domain-based lessons on organizing, cleaning, managing, and analyzing data, including ecology and geospatial curricula.

### Metadata and documentation

- [Metadata — UK Data Service](https://ukdataservice.ac.uk/learning-hub/research-data-management/document-your-data/metadata/) explains structured metadata and introduces standards such as Dublin Core, DDI, SDMX, and DataCite.
- [Data documentation: quantitative data — UK Data Service](https://ukdataservice.ac.uk/learning-hub/research-data-management/document-your-data/data-level/data-documentation-quantitative-data/) provides practical guidance on variable names, labels, definitions, units, code lists, missing values, and codebooks.
- [Study-level documentation — UK Data Service](https://ukdataservice.ac.uk/learning-hub/research-data-management/document-your-data/study-level-documentation/) describes the wider context needed to understand a dataset, including purpose, methods, coverage, processing, quality assurance, access, and licensing.

### FAIR principles

- Wilkinson, M. D. et al. (2016), [The FAIR Guiding Principles for scientific data management and stewardship](https://doi.org/10.1038/sdata.2016.18), is the foundational publication. Note that FAIR is a set of high-level principles rather than a specific technology or standard.
