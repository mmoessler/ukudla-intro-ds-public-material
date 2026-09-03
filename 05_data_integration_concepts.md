# Understand data-integration concepts

---

- Last Update: 2026-08-20
- Source: [05_data_integration_concepts.md](/learning-modules/intro-ds-module/05_data_integration_concepts.md)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Data management, acquisition, integration, and preparation](#data-management-acquisition-integration-and-preparation)
- [Begin with a data requirement](#begin-with-a-data-requirement)
- [Evaluate a source](#evaluate-a-source)
  - [Authority and origin](#authority-and-origin)
  - [Meaning and scope](#meaning-and-scope)
  - [Quality and revisions](#quality-and-revisions)
  - [Access and reproducibility](#access-and-reproducibility)
  - [Legal and ethical conditions](#legal-and-ethical-conditions)
- [Access methods](#access-methods)
  - [Downloadable files](#downloadable-files)
  - [APIs](#apis)
  - [Databases](#databases)
  - [Remote data services](#remote-data-services)
  - [Sensors and streams](#sensors-and-streams)
- [Tabular and semi-structured formats](#tabular-and-semi-structured-formats)
  - [CSV and TSV](#csv-and-tsv)
  - [Spreadsheets](#spreadsheets)
  - [JSON](#json)
  - [Parquet](#parquet)
- [Relational databases](#relational-databases)
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

[Why integrate data?](05_data_integration_motivation.md) establishes why source and alignment
decisions affect scientific results. This page gives you the vocabulary and
decision model required by [the maize and precipitation integration
application](05_data_integration_application.md).

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

Data management applies throughout the lifecycle. Acquisition selects and
retrieves evidence. Integration establishes relationships between sources.
Preparation handles analytical choices such as recoding, filtering,
missing-data treatment, and feature construction. A single script may perform
more than one activity, but its decisions should remain distinguishable.

---

## Begin with a data requirement

Do not begin by downloading every available file. First write what the project needs.

For example:

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

A precise requirement helps evaluate whether a source is suitable and prevents an acquisition script from becoming an undocumented collection of filters.

Record whether each condition is essential or preferred. No source may satisfy every preference.

---

## Evaluate a source

Ask the following questions.

### Authority and origin

- Who created or publishes the data?
- Is this the original provider or a republished copy?
- Is the collection method documented?

### Meaning and scope

- Does the provider measure the intended concept?
- Which populations, geographies, commodities, and periods are covered?
- Are values observed, reported, estimated, imputed, or modeled?

### Quality and revisions

- Are quality flags or uncertainty measures available?
- Are historical values revised?
- Is there a release schedule or version identifier?

### Access and reproducibility

- Is a stable file, API, or query interface available?
- Can the same subset be requested again?
- Are authentication, rate limits, cost, or availability constraints documented?

### Legal and ethical conditions

- What license and citation apply?
- May raw and derived data be redistributed?
- Are there privacy, confidentiality, or location-sensitivity risks?

The most convenient source is not necessarily the most authoritative or reproducible source.

---

## Access methods

### Downloadable files

A published file is a source snapshot. Record the exact URL or delivery method, access date, release, filename, size, and checksum.

Advantages:

- simple to inspect and preserve;
- often suitable for bulk data;
- can support offline work.

Risks:

- manual filters may be undocumented;
- providers may replace files at the same URL;
- large files can be inefficient;
- spreadsheet exports may mix data and presentation.

### APIs

An application programming interface accepts a request and returns a structured response.

A reproducible request records:

- base endpoint;
- path;
- parameters and values;
- pagination;
- API version;
- response format;
- authentication requirements;
- request date.

Always validate the response. An HTTP success status does not prove that the expected schema or complete result was returned.

### Databases

A database query selects rows and columns from one or more related tables. Record the database version or snapshot, connection target, SQL query, and relevant parameters.

Use read-only access for learning exercises when possible. Do not embed passwords in the script.

### Remote data services

Large climate and Earth-observation platforms often run filtering, aggregation, or computation near the hosted data.

Record:

- collection/product identifier and version;
- spatial and temporal filters;
- quality masks;
- scale, projection, and aggregation;
- exported result and job date;
- service-specific processing assumptions.

The exported table is derived data, not an untouched raw observation.

### Sensors and streams

Repeated observations may arrive continuously. Important metadata include device identifier, location, calibration, timestamp, time zone, sampling frequency, downtime, and quality status.

Streaming infrastructure is outside the core exercise, but the same provenance and validation principles apply.

---

## Tabular and semi-structured formats

### CSV and TSV

Delimited text is portable, but usually does not preserve:

- data types;
- variable definitions;
- units;
- missing-value rules;
- relationships between tables.

Record delimiter, encoding, decimal convention, date representation, and missing codes.

### Spreadsheets

Spreadsheets can contain multiple sheets, formulas, merged cells, notes, formatting, and several tables. Identify the sheet and cell range used. Avoid treating color or layout as machine-readable data.

### JSON

JSON can represent nested records and is common in APIs.

```json
{
  "country": {"code": "ZMB", "name": "Zambia"},
  "year": 2022,
  "indicator": {"code": "X", "value": 123.4}
}
```

Converting nested JSON to a table requires choices about which objects become rows and how nested fields are expanded. Record those choices.

### Parquet

Parquet preserves types and supports efficient analytical access. It is useful for larger derived tables but should not replace source metadata or provenance.

---

## Relational databases

Relational databases store data in tables connected through keys.

```text
country
  country_id ───────────────┐
  country_name              │
                            │
maize_observation           │
  country_id  <─────────────┘
  year
  value
```

Core concepts:

- a **primary key** identifies a record in its table;
- a **foreign key** refers to a record in another table;
- a query selects or combines records without copying the complete database;
- database state matters—a query can return different results after an update.

Example read-only query:

```sql
SELECT country_id, year, value
FROM maize_observation
WHERE year BETWEEN 1990 AND 2022;
```

Full database design, administration, permissions, transactions, and optimization are outside this introductory session.

---

## Spatial and temporal data

### Vector data

Vector data represent points, lines, or polygons. Common concerns include:

- stable geographic identifiers;
- coordinate reference system;
- boundary vintage;
- invalid geometry;
- geographic versus projected coordinates.

### Raster data

Raster data represent cells in a grid. Record:

- variable and unit;
- cell resolution;
- extent;
- coordinate reference system;
- time interval;
- missing or masked cells;
- processing level.

Summarizing raster cells to a country requires decisions about boundaries, weighting, partial cells, missing coverage, and time aggregation.

### Temporal data

Record whether a value refers to:

- an instant or interval;
- local time or UTC;
- day, month, season, calendar year, or growing year;
- a complete or partial period.

Never infer a time zone or reporting period only from the display format.

---

## Grain, keys, and join relationships

State the grain of every input before joining. For example:

```text
FAOSTAT table: one row per country–year–item–element–unit
CHIRPS table: one row per country–October–April season
Target table: one row per country–year
```

The input tables must first be expressed at grains compatible with the target.
A join relationship describes how many rows on one side can match a key on the
other:

| Relationship | Meaning | Expected row behavior |
| --- | --- | --- |
| One-to-one | At most one matching row on each side | Keys should not multiply |
| One-to-many | One left row can match several right rows | Left observations can expand |
| Many-to-one | Several left rows can match one right row | Right values can repeat legitimately |
| Many-to-many | Several rows can match on both sides | Often produces multiplication and requires explicit justification |

The relationship is a property of the data at the stated grain, not merely an
argument supplied to software. Test key uniqueness on both sides before the
join. Unexpected many-to-many matches usually indicate an incomplete key or
incompatible grain.

---

## Identifiers and crosswalks

Provider labels are designed for display and can differ in spelling, language,
punctuation, abbreviation, or historical meaning. Prefer stable source codes
when available.

A **crosswalk** maps source-specific identifiers to a reviewed project
identifier:

```csv
project_country_id,project_country_name,faostat_area_label,valid_from,valid_to,note
ZAF,South Africa,South Africa,1990,2022,Reviewed country mapping
ZMB,Zambia,Zambia,1990,2022,Reviewed country mapping
```

Treat the crosswalk as data:

- retain both source identifiers and readable labels;
- test uniqueness in the direction required by the join;
- record geographic validity periods and boundary changes;
- separate countries from aggregates and regions;
- preserve unresolved mappings instead of guessing;
- review fuzzy matches manually before accepting them.

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

Alignment can require aggregation, conversion, or mapping. Each operation can
discard information or change interpretation, so record the rule and retain
the original variables or metadata needed to audit it.

---

## Join choice and integration audits

Choose a join from the intended study population:

- `left_join()` retains every key from the designated primary source;
- `inner_join()` retains only keys observed in both sources;
- `full_join()` retains the union and is useful for diagnosing overlap;
- `anti_join()` reports keys present on only one side.

No join is universally safest. An inner join may silently remove countries or
years; a left join may introduce missing complementary values; a full join may
include records outside the target population.

Before joining, record the expected relationship, unmatched keys, and row-count
effect. Afterwards, audit:

- input and output row counts;
- unmatched keys in both directions;
- duplicate or multiplied output keys;
- missingness introduced by integration;
- geographic and temporal coverage;
- whether the output still has the intended grain.

Do not suppress an unexpected relationship warning merely to obtain output.

---

## Lineage

Lineage connects each output variable to its origin and transformations. A
compact lineage table can record:

| Output variable | Source artifact | Source field | Operation | Unit |
| --- | --- | --- | --- | --- |
| `yield_tonnes_per_hectare` | FAOSTAT extract | `value` where `element = Yield` | Reshape and convert from kg/ha | `t/ha` |
| `growing_season_precipitation_mm` | CHIRPS snapshot | Daily spatial averages | Sum October–April and join by country and ending year | `mm` |

Also record input checksums or releases, acquisition scripts, crosswalk version,
aggregation and conversion rules, integration script, output checksum, and
known information loss. Provenance identifies the inputs; lineage explains how
they became the integrated artifact.

---

## Plan for changing services

External sources can change their:

- endpoint and authentication;
- schema and variable names;
- classifications and geographic codes;
- historical values;
- rate limits and availability;
- license and access rules.

Make a workflow more resilient by:

- requesting a documented version or release;
- recording the access date and complete query;
- preserving the raw response when permitted;
- recording file size and checksum;
- validating schema and coverage immediately;
- caching responses responsibly;
- providing a fixed teaching snapshot and instructions for refreshing it.

Do not automatically fall back to a different provider. That changes the evidence and requires a scientific decision.

---

## Credentials and responsible access

- Never commit API keys, tokens, passwords, or private certificates.
- Read secrets from environment variables or an ignored local configuration file.
- Provide an example configuration containing names but no secret values.
- Request only the permissions required.
- Respect rate limits, terms, and provider infrastructure.
- Remove credentials from logs and error reports.
- Revoke and rotate a credential if it is exposed.

A deleted secret may remain in Git history. Prevention is safer than removing it later.

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

The record should allow another person to understand exactly what was requested, even if the service later changes.

---

## Check your understanding

1. Why should a data requirement be written before selecting a source?
2. What must be recorded to reproduce an API request?
3. Why can a successful API response still be an acquisition failure?
4. Which information can CSV fail to preserve?
5. Why is a country summary exported from a raster service derived rather than raw data?
6. How should credentials be supplied to a script?

7. What distinguishes a one-to-many join from an accidental many-to-many join?
8. Why should unmatched keys be inspected from both directions?
9. Which information belongs in a country crosswalk rather than an inline code replacement?

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
data](05_data_integration_application.md). The application turns managed FAOSTAT and CHIRPS
inputs, grain and key expectations, a reviewed crosswalk, explicit joins,
audit tables, and lineage into inspectable project artifacts.
