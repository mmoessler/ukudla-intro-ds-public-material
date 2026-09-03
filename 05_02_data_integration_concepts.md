# 5.2) Understand data-integration concepts

---

- Last Update: 2026-08-20
- Source: [05_02_data_integration_concepts.md](/learning-modules/intro-ds-module/05_02_data_integration_concepts.md)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Data management, acquisition, integration, and preparation](#data-management-acquisition-integration-and-preparation)
- [Begin with a data requirement](#begin-with-a-data-requirement)
- [Evaluate a source](#evaluate-a-source)
- [Access methods](#access-methods)
  - [Downloadable files](#downloadable-files)
  - [APIs](#apis)
  - [Databases](#databases)
  - [Remote data services](#remote-data-services)
- [Tabular and semi-structured formats](#tabular-and-semi-structured-formats)
  - [CSV and TSV](#csv-and-tsv)
  - [Spreadsheets](#spreadsheets)
  - [JSON](#json)
  - [Parquet](#parquet)
- [Spatial and temporal data](#spatial-and-temporal-data)
  - [Vector data](#vector-data)
  - [Raster data](#raster-data)
  - [Temporal data](#temporal-data)
- [Grain, keys, and join relationships](#grain-keys-and-join-relationships)
- [Identifiers and crosswalks](#identifiers-and-crosswalks)
- [Alignment across sources](#alignment-across-sources)
- [Join choice and integration audits](#join-choice-and-integration-audits)
- [Lineage](#lineage)
- [Plan for changing services](#plan-for-changing-services)
- [Credentials and responsible access](#credentials-and-responsible-access)
- [Source and acquisition record](#source-and-acquisition-record)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
  - [Acquisition and responsible access](#acquisition-and-responsible-access)
  - [Integration and joins](#integration-and-joins)
  - [Spatial and temporal integration](#spatial-and-temporal-integration)
- [Continue to Application](#continue-to-application)

---

## Learning objectives

After completing this page, you should be able to:

- translate an information gap into a requirement for an additional source;
- evaluate authority, coverage, documentation, quality, license, and reproducibility;
- compare file, API, database, and remote-service access;
- recognize common tabular, JSON, relational, vector, raster, and temporal representations;
- state source grains, keys, and expected join relationships;
- explain why crosswalks and temporal, spatial, unit, and classification alignment are analytical artifacts;
- choose a join from the intended population and audit its result;
- anticipate service changes and credential risks; and
- create a complete source and acquisition record without confusing it with an integration audit.

---

## Place in the session

This is the **Concepts** part of the Data Integration session:

```text
Motivation  →  Concepts  →  Application
                ↑
             this page
```

[Why integrate data?](05_01_data_integration_motivation.md) establishes why source and alignment
decisions affect scientific results. This page gives you the vocabulary and
decision model required by [the maize and precipitation integration
application](05_03_data_integration_application.md).

Use the concepts as questions:

- What evidence does the project require?
- Why is this source suitable, and can the request be reconstructed?
- What does one row in each source represent?
- Which identifiers, periods, spaces, units, and classifications must align?
- What row-count and matching behavior should the join produce?
- Can every output variable be traced back to an input and operation?

---

## Data management, acquisition, integration, and preparation

These activities overlap, but emphasize different questions:

| Activity | Central question | Typical output |
| --- | --- | --- |
| Data management | How are data made understandable, trustworthy, organized, and auditable? | Managed source and derived artifacts |
| Acquisition | How was additional evidence obtained and identified? | Preserved response or fixed source snapshot |
| Integration | How were compatible observations connected across sources? | Audited multi-source interim dataset |
| Preparation | How were values adapted for a particular analysis? | Analysis-ready dataset and derived variables |

Data management applies throughout the lifecycle; acquisition retrieves
evidence, integration connects sources, and preparation handles choices such
as recoding, filtering, and feature construction. A single script may perform
more than one activity, but its decisions should stay distinguishable.

---

## Begin with a data requirement

Do not begin by downloading every available file. First write what the
project needs, for example:

```text
Concept: annual maize yield
Entities: selected Southern African countries
Period: 1990–2022
Frequency: annual
Geographic level: country
Measurement: provider-defined yield and unit
Quality information: retain provider flags
Required metadata: definitions, methods, codes, license, release
```

A precise requirement helps evaluate source suitability and prevents an
acquisition script from becoming an undocumented collection of filters.
Record whether each condition is essential or preferred.

---

## Evaluate a source

| Dimension | Questions to ask |
| --- | --- |
| Authority and origin | Who created or publishes the data? Is this the original provider or a republished copy? Is the collection method documented? |
| Meaning and scope | Does the provider measure the intended concept? Which populations, geographies, commodities, and periods are covered? Are values observed, reported, estimated, imputed, or modeled? |
| Quality and revisions | Are quality flags or uncertainty measures available? Are historical values revised? Is there a release schedule or version identifier? |
| Access and reproducibility | Is a stable file, API, or query interface available? Can the same subset be requested again? Are authentication, rate limits, cost, or availability constraints documented? |
| Legal and ethical conditions | What license and citation apply? May raw and derived data be redistributed? Are there privacy, confidentiality, or location-sensitivity risks? |

The most convenient source is not necessarily the most authoritative or
reproducible one.

---

## Access methods

### Downloadable files

A published file is a source snapshot. Record its exact URL or delivery
method, access date, release, filename, size, and checksum. Files are simple
to inspect, preserve, and use offline, but providers may replace them at the
same URL and spreadsheet exports may mix data with presentation.

### APIs

An application programming interface accepts a request and returns a
structured response. A reproducible request records the endpoint,
parameters, pagination, API version, response format, authentication, and
request date. Always validate the response — a success status does not prove
the expected schema or complete result was returned.

### Databases

A database query selects rows and columns from related tables: a **primary
key** identifies a record in its table, a **foreign key** refers to a record
in another table, and results can change after an update because database
state matters. Record the database version, connection target, SQL query, and
parameters; use read-only access for learning exercises and never embed
passwords in the script. Full database design, administration, and
optimization are outside this introductory session.

### Remote data services

Large climate and Earth-observation platforms often filter, aggregate, or
compute near the hosted data. Record the collection identifier and version,
spatial/temporal filters, quality masks, projection, aggregation rule, and
export date — the exported table is derived data, not an untouched raw
observation. Continuous sensor and stream sources carry similar metadata
(device, location, calibration, timestamp, frequency) but sit outside this
course's core exercise.

---

## Tabular and semi-structured formats

### CSV and TSV

Delimited text is portable but usually does not preserve data types,
definitions, units, missing-value rules, or relationships between tables.
Record the delimiter, encoding, decimal convention, date representation, and
missing codes.

### Spreadsheets

Spreadsheets can contain multiple sheets, formulas, merged cells, notes, and
several tables. Identify the sheet and cell range used, and avoid treating
color or layout as machine-readable data.

### JSON

JSON can represent nested records and is common in APIs, for example a
`country` object nested inside a `year`/`indicator` record. Converting nested
JSON to a table requires choices about which objects become rows and how
nested fields are expanded. Record those choices.

### Parquet

Parquet preserves types and supports efficient analytical access for larger
derived tables, but it should not replace source metadata or provenance.

---

## Spatial and temporal data

### Vector data

Vector data represent points, lines, or polygons. Common concerns include
stable geographic identifiers, coordinate reference system, boundary vintage,
invalid geometry, and geographic versus projected coordinates.

### Raster data

Raster data represent cells in a grid. Record the variable and unit, cell
resolution, extent, coordinate reference system, time interval, missing or
masked cells, and processing level. Summarizing raster cells to a country
requires decisions about boundaries, weighting, and coverage.

### Temporal data

Record whether a value refers to an instant or interval, local time or UTC,
day, month, season, calendar year, or growing year, and a complete or partial
period. Never infer a time zone or reporting period only from the display
format.

---

## Grain, keys, and join relationships

State the grain of every input before joining. For example:

```text
FAOSTAT table: one row per country–year–item–element–unit
CHIRPS table: one row per country–October–April season
Target table: one row per country–year
```

The input tables must first be expressed at grains compatible with the
target. A join relationship describes how many rows on one side can match a
key on the other:

| Relationship | Meaning | Expected row behavior |
| --- | --- | --- |
| One-to-one | At most one matching row on each side | Keys should not multiply |
| One-to-many | One left row can match several right rows | Left observations can expand |
| Many-to-one | Several left rows can match one right row | Right values can repeat legitimately |
| Many-to-many | Several rows can match on both sides | Often produces multiplication and requires explicit justification |

The relationship is a property of the data at the stated grain, not an
argument supplied to software. Test key uniqueness on both sides before
joining — an unexpected many-to-many match usually signals an incomplete key
or incompatible grain.

---

## Identifiers and crosswalks

Provider labels are designed for display and can differ in spelling,
language, punctuation, abbreviation, or historical meaning. Prefer stable
source codes when available.

A **crosswalk** maps source-specific identifiers to a reviewed project
identifier:

```csv
project_country_id,project_country_name,faostat_area_label,valid_from,valid_to,note
ZAF,South Africa,South Africa,1990,2022,Reviewed country mapping
```

Treat the crosswalk as data: retain both identifiers and readable labels,
test uniqueness in the join direction, record validity periods and boundary
changes, and review fuzzy matches manually rather than guessing.

---

## Alignment across sources

Matching identifiers is only one part of integration:

| Dimension | Question to resolve |
| --- | --- |
| Concept | Do the variables measure compatible phenomena? |
| Grain | Do rows represent compatible observational units? |
| Time | Calendar year, growing season, interval, instant, or partial period? |
| Space | Which boundary vintage, resolution, CRS, and aggregation rule apply? |
| Unit | Are definitions and dimensions compatible before conversion? |
| Classification | Do commodity, land-use, or status categories correspond? |
| Quality | Which flags, uncertainty measures, or source statuses must remain visible? |

Alignment can require aggregation, conversion, or mapping; each operation can
discard information or change interpretation, so record the rule and retain
what's needed to audit it.

---

## Join choice and integration audits

Choose a join from the intended study population:

- `left_join()` retains every key from the designated primary source;
- `inner_join()` retains only keys observed in both sources;
- `full_join()` retains the union and is useful for diagnosing overlap;
- `anti_join()` reports keys present on only one side.

No join is universally safest. An inner join may silently remove countries or
years; a left join may introduce missing complementary values; a full join
may include records outside the target population.

Before joining, record the expected relationship, unmatched keys, and
row-count effect. Afterwards, audit row counts, unmatched keys in both
directions, duplicate output keys, missingness, and coverage, and confirm the
output still has the intended grain. Do not suppress an unexpected
relationship warning merely to obtain output.

---

## Lineage

Lineage connects each output variable to its origin and transformations. A
compact lineage table can record:

| Output variable | Source artifact | Source field | Operation | Unit |
| --- | --- | --- | --- | --- |
| `yield_tonnes_per_hectare` | FAOSTAT extract | `value` where `element = Yield` | Reshape and convert from kg/ha | `t/ha` |
| `growing_season_precipitation_mm` | CHIRPS snapshot | Daily spatial averages | Sum October–April and join by country and ending year | `mm` |

Also record input checksums, acquisition and integration scripts, crosswalk
version, conversion rules, output checksum, and known information loss —
provenance identifies the inputs, lineage explains how they became the
integrated artifact.

---

## Plan for changing services

External sources can change their endpoint and authentication, schema and
variable names, classifications and geographic codes, historical values, rate
limits and availability, and license and access rules.

Make a workflow more resilient: request a documented version, record the
access date and complete query, preserve the raw response when permitted,
validate schema and coverage immediately, and provide a fixed teaching
snapshot with refresh instructions. Do not automatically fall back to a
different provider — that changes the evidence and requires a scientific
decision.

---

## Credentials and responsible access

Never commit API keys, tokens, passwords, or certificates. Read secrets from
environment variables or an ignored local config file, request only the
permissions required, and keep them out of logs and error reports. Revoke and
rotate any exposed credential — a deleted secret may remain in Git history, so
prevention is safer than cleanup.

---

## Source and acquisition record

For each source, record:

```yaml
source_id: faostat_maize
provider: "provider name"
dataset: "exact dataset or product"
version: "release or product version"
accessed: "YYYY-MM-DD"
access_method: "file, API, database, or remote service"
location: "URL, endpoint, database, or collection ID"
parameters: "countries, years, variables, filters"
license: "license or terms"
raw_artifact: "project path"
checksum_sha256: "checksum when applicable"
script: "acquisition script"
notes: "limitations and fallback snapshot"
```

The record should allow another person to understand exactly what was
requested, even if the service later changes.

---

## Check your understanding

1. Why should a data requirement be written before selecting a source?
2. What must be recorded to reproduce an API request?
3. Which information can CSV fail to preserve?
4. How should credentials be supplied to a script?
5. What distinguishes a one-to-many join from an accidental many-to-many join?
6. Why should unmatched keys be inspected from both directions?

---

## Further resources

### Acquisition and responsible access

- [Research Data Management — The Turing Way](https://book.the-turing-way.org/reproducible-research/rdm/) connects acquisition, documentation, storage, sharing, and preservation.
- [HTTP overview — MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) explains the request–response model underlying web APIs.
- [World Bank Indicators API documentation](https://datahelpdesk.worldbank.org/knowledgebase/topics/125589-developer-information) documents a public indicator service suitable for practising reproducible requests.

### Integration and joins

- Wickham, H., Çetinkaya-Rundel, M., and Grolemund, G., [R for Data Science (2e): Joins](https://r4ds.hadley.nz/joins.html), covers keys, mutating joins, filtering joins, and relationship diagnostics.
- [Mutating joins — dplyr reference](https://dplyr.tidyverse.org/reference/mutate-joins.html) documents join arguments, unmatched rows, and relationship checks.
- [Data Wrangling with dplyr — Data Carpentry](https://datacarpentry.github.io/r-socialsci/instructor/03-dplyr.html) provides a practical introduction to relational operations in R.

### Spatial and temporal integration

- [Geocomputation with R](https://r.geocompx.org/) introduces vector, raster, coordinate-reference, and spatial aggregation concepts.
- [CF Conventions](https://cfconventions.org/) document metadata conventions widely used for climate and forecast data in NetCDF.

---

## Continue to Application

Continue with [Integrate maize-yield and precipitation
data](05_03_data_integration_application.md). The application turns managed FAOSTAT and CHIRPS
inputs, grain and key expectations, a reviewed crosswalk, explicit joins,
audit tables, and lineage into inspectable project artifacts.
