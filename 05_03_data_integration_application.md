# 5.3) Integrate maize-yield and precipitation data

---

- Last Update: 2026-08-20
- Source: [05_03_data_integration_application.md](/learning-modules/intro-ds-module/05_03_data_integration_application.md)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Scenario and deliverable](#scenario-and-deliverable)
- [1. Identify the information gap](#1-identify-the-information-gap)
  - [Why CHIRPS is a reasonable choice](#why-chirps-is-a-reasonable-choice)
- [2. Understand and validate each source](#2-understand-and-validate-each-source)
  - [FAOSTAT maize data](#faostat-maize-data)
  - [CHIRPS precipitation data](#chirps-precipitation-data)
- [3. Define the integration contract](#3-define-the-integration-contract)
- [4. Align identifiers, space, and time](#4-align-identifiers-space-and-time)
  - [Country identifiers](#country-identifiers)
  - [Spatial support](#spatial-support)
  - [Temporal support](#temporal-support)
- [5. Integrate and audit the sources](#5-integrate-and-audit-the-sources)
- [6. Document and interpret the result](#6-document-and-interpret-the-result)
- [Extending the integration](#extending-the-integration)
- [Troubleshooting](#troubleshooting)
  - [The maize panel is missing](#the-maize-panel-is-missing)
  - [The CHIRPS checksum fails](#the-chirps-checksum-fails)
  - [A country label is unmatched](#a-country-label-is-unmatched)
  - [Rows multiply](#rows-multiply)
  - [Years appear shifted](#years-appear-shifted)
- [Completion checklist](#completion-checklist)
- [Reflect on the application](#reflect-on-the-application)
- [Further resources](#further-resources)

---

## Learning objectives

After completing this exercise, you should be able to:

- explain why precipitation is added to the managed maize dataset;
- state the grain and candidate key of both sources and the result;
- distinguish source acquisition from the central integration decisions;
- explain the country, spatial, and temporal alignment;
- state a join relationship and predict its effect on row count;
- audit keys, coverage, duplicates, matches, and missing values; and
- trace integrated variables to their sources and transformations.

---

## Place in the session

This is the **Application** part of the Data Integration session:

~~~text
Motivation  →  Concepts  →  Application
                              ↑
                           this page
~~~

The preceding Data Management topic made the fixed FAOSTAT input
understandable and auditable. This application applies the same practices
again while extending the project with CHIRPS precipitation.

Review [Why integrate data?](05_01_data_integration_motivation.md) and [Understand data-integration
concepts](05_02_data_integration_concepts.md) before beginning.

---

## Scenario and deliverable

FAOSTAT describes annual maize yield, production, and harvested area, but it
does not describe environmental conditions associated with each country-year.
The project adds CHIRPS precipitation as a plausible environmental variable.

The tracked inputs are:

~~~text
data/input/faostat-maize-yield-sample.csv
data/input/chirps-growing-season-precipitation.csv
~~~

The workflow produces:

~~~text
data/derived/maize-yield-panel.csv
data/derived/maize-yield-with-precipitation.csv
results/tables/data-integration-audit.csv
~~~

The result is an analysis input, not evidence that precipitation causes
differences in maize yield.

---

## 1. Identify the information gap

Begin with the analytical need:

> We want to explore whether wetter or drier growing seasons coincide with
> differences in reported country-level maize yield.

Record:

- the concept needed: growing-season precipitation;
- the existing analytical unit: country-year;
- desired coverage: nine project countries, 1990–2022;
- expected unit: millimetres;
- acceptable spatial and temporal approximations; and
- limitations that would make a source unsuitable.

CHIRPS provides long-running daily gridded precipitation estimates. Its
suitability still depends on how those estimates are summarized and aligned.

### Why CHIRPS is a reasonable choice

Applying the evaluation questions from the concepts page:

- **Authority**: CHIRPS is produced by the Climate Hazards Center (UC Santa
  Barbara) with USGS, a documented, non-commercial climate data provider.
- **Meaning and scope**: it estimates rainfall, not soil moisture or crop
  water stress — a proxy for growing-season moisture, not a direct yield
  driver.
- **Quality**: it blends satellite infrared estimates with sparse station
  observations, so accuracy varies with local station density.
- **Access**: available as a stable gridded product through ClimateSERV and
  direct download, supporting repeated, versioned requests.
- **License**: public domain with a citation requirement.

A station-based rainfall network or a reanalysis product such as ERA5 could
serve the same role, with different coverage, resolution, and access
trade-offs. The choice of source is a recorded scientific decision, not a
default.

Acquisition supports this step, but is not the central learning objective.
The normal student workflow uses fixed, checksummed snapshots and works
offline. Maintainers deliberately refresh them using the acquisition scripts.

---

## 2. Understand and validate each source

### FAOSTAT maize data

The tracked FAOSTAT input has this grain:

> One row represents one maize element for one reporting area, calendar year,
> and unit.

<code>scripts/prepare-maize-data.R</code> validates element-unit combinations,
reshapes the values, converts yield from <code>kg/ha</code> to
<code>t/ha</code>, and creates one row per country and year.

Before trusting the input, inspect it directly:

~~~r
maize_raw <- readr::read_csv("data-raw/faostat-maize-yield-sample.csv")
dplyr::count(maize_raw, area, element, unit)
~~~

This surfaces which country, element, and unit combinations are actually
present before any reshaping happens — a mismatch here explains a downstream
validation failure more directly than reading the validation report alone.

### CHIRPS precipitation data

The tracked CHIRPS input has this grain:

> One row represents one project country and one October–April season.

Its candidate key is:

~~~text
project_country_id + year
~~~

Validate that it contains required columns, nine known project identifiers,
all years from 1990 through 2022, unique keys, non-negative precipitation, and
212 or 213 daily observations per season.

The source descriptions, dictionaries, provenance, and human-readable dataset
pages should be reviewed before integration. The integration inherits the
limitations of all inputs.

A parallel check confirms the expected season coverage:

~~~r
chirps_raw <- readr::read_csv("data-raw/chirps-growing-season-precipitation.csv")
dplyr::count(chirps_raw, project_country_id) |> dplyr::filter(n != 33)
~~~

An empty result confirms that every country has all 33 seasons (1990 through
2022).

---

## 3. Define the integration contract

The target grain is:

> One row represents one project country and year, containing maize measures
> and precipitation for the October–April season ending in that year.

The target key is <code>project_country_id + year</code>.

| Property | Expectation |
| --- | --- |
| Left table | Prepared FAOSTAT maize panel |
| Right table | CHIRPS growing-season precipitation |
| Join key | Project country identifier and year |
| Relationship | One-to-one |
| Join type | Left join |
| Expected left rows | 297 |
| Expected output rows | 297 |
| Expected unmatched maize keys | 0 |
| Expected duplicate output keys | 0 |

This contract turns assumptions about the join into testable conditions. A
join function cannot determine whether the contract is scientifically valid.

---

## 4. Align identifiers, space, and time

### Country identifiers

The FAOSTAT input contains provider labels; CHIRPS uses stable project
identifiers derived from Natural Earth <code>ADM0_A3</code> values. The
reviewed mapping is stored in:

~~~text
metadata/project-country-crosswalk.csv
~~~

It maps provider labels to project identifiers and records validity periods
and naming notes. A crosswalk is a governed dataset, not an invisible string
correction in code.

### Spatial support

CHIRPS begins as daily gridded precipitation. ClimateSERV calculates the
spatial average across each polygon in
<code>metadata/project-country-boundaries.geojson</code>. These generalized
Natural Earth country polygons are not precise maize-growing areas.

### Temporal support

The acquisition workflow sums daily spatial averages from 1 October through
30 April. A season is assigned to the year in which it ends:

~~~text
year = 2022
season_start_date = 2021-10-01
season_end_date = 2022-04-30
~~~

The result, <code>growing_season_precipitation_mm</code>, is a seasonal sum of
country-area daily averages. It is not a station measurement or a
maize-area-weighted estimate.

---

## 5. Integrate and audit the sources

From the example-project root, run:

~~~bash
Rscript scripts/prepare-maize-data.R
Rscript scripts/integrate-data.R
~~~

The integration script:

1. verifies the tracked CHIRPS checksum;
2. validates its schema, keys, values, and coverage;
3. maps FAOSTAT labels through the country crosswalk;
4. checks uniqueness in both sources;
5. performs a declared one-to-one left join; and
6. refuses to write output if row count or key uniqueness changes.

A simplified join is:

~~~r
integrated <- maize_with_id |>
  dplyr::left_join(
    precipitation,
    by = c("project_country_id", "year"),
    relationship = "one-to-one"
  )
~~~

Inspect <code>results/tables/data-integration-audit.csv</code>. It records
source and output rows, duplicate output keys, unmatched keys, and missing
precipitation. During development, use anti-joins to inspect unmatched keys
directly. Never treat an empty or non-empty anti-join as self-explanatory.

A passing audit looks like:

| Check | Left rows | Right rows | Output rows | Unmatched left | Unmatched right | Duplicate keys |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| maize + precipitation | 297 | 297 | 297 | 0 | 0 | 0 |

If unmatched left keys appear, the most likely cause is a country missing
from the crosswalk or absent from a CHIRPS season. If the output row count
exceeds 297, the relationship is not truly one-to-one, and the script should
stop rather than silently multiply rows.

---

## 6. Document and interpret the result

The new artifact needs its own documentation:

- human-readable description:
  <code>docs/data/maize-yield-with-precipitation.md</code>;
- column definitions:
  <code>metadata/maize-yield-with-precipitation-data-dictionary.csv</code>;
- transformation lineage in <code>metadata/provenance.yml</code>;
- source descriptions in <code>metadata/source-metadata.yml</code>; and
- executable transformation in <code>scripts/integrate-data.R</code>.

For example, the data dictionary should describe
<code>growing_season_precipitation_mm</code> with its unit, definition, and
season boundary, not only its column name:

~~~csv
variable,label,definition,type,unit
growing_season_precipitation_mm,Growing-season precipitation,Summed October-April daily spatial-average precipitation for the ending year,double,mm
~~~

A reader who sees only the derived table cannot recover the season boundary
or the aggregation method from the column name alone; the dictionary and
provenance record carry that meaning forward.

The table can describe associations between national maize statistics and
country-area precipitation. It cannot establish causation. Rainfall timing,
irrigation, heat, soils, varieties, pests, inputs, management, reporting
differences, and subnational production patterns are omitted. Both
insufficient and excessive rainfall can reduce yield.

---

## Extending the integration

The same workflow generalizes to other environmental variables. Adding a
second source — for example mean growing-season temperature — repeats the
same sequence: define the requirement, evaluate the source, define a new
integration contract with its own expected relationship, align identifiers
and time, and add a second audit row. It does not require a new integration
pattern.

A more demanding extension would use subnational or maize-growing-area
weighted precipitation rather than a country-polygon average. That would
change the meaning of <code>growing_season_precipitation_mm</code> from a
coarse country-wide proxy to an estimate more directly tied to where maize is
actually grown, at the cost of a more complex spatial aggregation step and a
new source to evaluate and document.

---

## Troubleshooting

### The maize panel is missing

Run <code>scripts/prepare-maize-data.R</code> before integration.

### The CHIRPS checksum fails

Do not bypass the check. Compare the file with its record in
<code>metadata/provenance.yml</code>.

### A country label is unmatched

Review <code>metadata/project-country-crosswalk.csv</code>. Do not apply an
ad hoc string replacement.

### Rows multiply

Stop and inspect both keys and the declared relationship. Do not remove
duplicates until their meaning is understood.

### Years appear shifted

Review the October–April definition. The join uses the year in which the
season ends.

---

## Completion checklist

- [ ] The information gap and reason for adding precipitation are stated.
- [ ] Both source grains and candidate keys are tested.
- [ ] Country identifiers use the reviewed crosswalk.
- [ ] Spatial and temporal aggregation decisions are explicit.
- [ ] The expected join relationship and row count are stated.
- [ ] Duplicate, unmatched, and missing records are audited.
- [ ] The integrated artifact has a dictionary, lineage, and readable documentation.
- [ ] Scientific limitations are reported without claiming causation.
- [ ] Fixed source snapshots remain unchanged.

---

## Reflect on the application

1. Why is precipitation a reasonable addition to the maize data?
2. What does one CHIRPS row represent after aggregation?
3. Why is the project identifier preferable to country-name matching?
4. Which alignment decision most affects scientific meaning?
5. What would change if rainfall were weighted to maize-growing areas?
6. Does a complete one-to-one join establish conceptual compatibility?
7. Which data-management practices are repeated after integration?
8. Which additional source would you consider, and what new mismatch would it introduce?

---

## Further resources

- [R for Data Science (2e): Joins](https://r4ds.hadley.nz/joins.html)
- [Mutating joins — dplyr reference](https://dplyr.tidyverse.org/reference/mutate-joins.html)
- [CHIRPS overview](https://www.chc.ucsb.edu/data/chirps)
- [ClimateSERV API documentation](https://climateserv.servirglobal.net/develop-api)
- [Geocomputation with R](https://r.geocompx.org/)
- [The Turing Way: Research Data Management](https://book.the-turing-way.org/reproducible-research/rdm/)
