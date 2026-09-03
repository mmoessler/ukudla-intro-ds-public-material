# 4.4) Apply data management: maize-yield worked example

---

- Source: [04_04_data_management_application.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/04_04_data_management_application.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/04_04_data_management_application.md)
- Feedback: [Topic 04: Data Management](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/5)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Scenario and deliverable](#scenario-and-deliverable)
  - [Transfer the workflow to another study design](#transfer-the-workflow-to-another-study-design)
- [Before you begin](#before-you-begin)
- [1. Preserve and identify the raw input](#1-preserve-and-identify-the-raw-input)
- [2. Establish the dataset grain](#2-establish-the-dataset-grain)
- [3. Create a data dictionary](#3-create-a-data-dictionary)
- [4. Record provenance](#4-record-provenance)
- [5. Organize and govern the project artifacts](#5-organize-and-govern-the-project-artifacts)
- [6. Validate structure](#6-validate-structure)
  - [Required columns](#required-columns)
  - [Coverage](#coverage)
  - [Candidate-key uniqueness](#candidate-key-uniqueness)
  - [Allowed categories](#allowed-categories)
- [7. Validate meaning](#7-validate-meaning)
  - [Missingness](#missingness)
  - [Ranges](#ranges)
  - [Flags](#flags)
  - [Internal consistency](#internal-consistency)
- [8. Report rather than silently repair](#8-report-rather-than-silently-repair)
- [Troubleshooting](#troubleshooting)
  - [Problem 01: Column names differ from the example](#problem-01-column-names-differ-from-the-example)
  - [Problem 02: The candidate key is not unique](#problem-02-the-candidate-key-is-not-unique)
  - [Problem 03: Numbers were imported as text](#problem-03-numbers-were-imported-as-text)
  - [Problem 04: The checksum differs from another learner's](#problem-04-the-checksum-differs-from-another-learners)
  - [Problem 05: A validation check stops the script](#problem-05-a-validation-check-stops-the-script)
- [Completion checklist](#completion-checklist)
- [Reflect on the application](#reflect-on-the-application)
- [Further resources](#further-resources)
  - [Practical workflow and checks](#practical-workflow-and-checks)
  - [Organization and documentation](#organization-and-documentation)
  - [Working with tabular data in R](#working-with-tabular-data-in-r)

---

## Learning objectives

After completing this exercise, you should be able to:

- inspect a source file without changing it;
- state and test its expected grain and key;
- build a project data dictionary and provenance record;
- organize project artifacts and make justified storage and sharing decisions;
- implement structural and semantic checks in R;
- distinguish a failed validation from a cleaning decision; and
- communicate unresolved data-quality concerns.

---

## Place in the session

This is the **Application** part of the Data Management session:

```text
Motivation  →  Concepts  →  Application
                              ↑
                           this page
```

Before beginning, review [Why manage data?](04_01_data_management_motivation.md) and [Understand data-management concepts](04_02_data_management_concepts.md). The exercise assumes you can explain grain, candidate keys, metadata, a data dictionary, provenance, organization, and fitness for purpose.

The numbered steps apply those concepts. Do not continue past a failed expectation to reach the end of the instructions — inspect the evidence, explain the failure, and record what must be resolved.

---

## Scenario and deliverable

> **Worked-example scope:** The numbered instructions use a secondary FAOSTAT
> dataset so that every learner can reproduce the same result. The method is
> defined by its decisions and evidence, not by this source. Use the transfer
> table below when applying it to data collected in another study design.

You have received a fixed FAOSTAT teaching extract of maize production, harvested area, and yield for selected Southern African countries. Before anyone prepares, visualizes, or models these values, the project team must determine what they mean and whether they match the expected structure.

Your task is to create an inspectable data-management package:

```text
maize-yield-project/
├── data/
│   └── input/
│       └── faostat-maize-yield-sample.csv  # supplied input; unchanged
├── metadata/
│   ├── faostat-maize-yield-data-dictionary.csv
│   └── provenance.yml          # source and history of the input
├── scripts/
│   └── validate-data.R         # repeatable expectations and checks
└── reports/
    └── data-validation.html    # findings, limitations, and status
```

### Transfer the workflow to another study design

| Project context | Define as the managed input | Record in metadata and provenance | Validate before use |
| --- | --- | --- | --- |
| Laboratory | Instrument export or assay result | Sample lineage, protocol, instrument, batch, calibration, detection limits | Sample IDs, units, controls, flags, ranges, replicate structure |
| Field experiment | Plot- or unit-level observations | Design, treatment assignment, blocks, protocol deviations, collection dates | Randomization structure, treatment codes, plot IDs, missing plots, measurement ranges |
| Field observation | Farm, plot, organism, or visit records | Sampling frame, recruitment or site selection, measurement protocol, consent and sensitivity | Coverage, repeated visits, selection, identifiers, missingness, plausible values |
| Secondary data | Preserved provider snapshot | Provider, version, access method, license, revisions, definitions | File identity, provider codes, coverage, flags, keys, units |

In every context, preserve the received or collected input, define its grain
and key, document how values were produced, validate fitness for the intended
question, and report unresolved concerns without silently repairing them.

The package is complete when another person can determine:

- what one row and each variable represent;
- where the data came from and whether the exact input changed;
- which structural and semantic expectations were checked;
- which checks passed, warned, failed, or remain unknown;
- which license, citation, access, and sharing conditions apply;
- which decisions are deferred to Data Preparation.

---

## Before you begin

Use the supplied maize-yield project and fixed FAOSTAT teaching extract. Begin at the project root:

```bash
pwd
git status
ls data-raw metadata scripts reports
```

The exercise assumes that the raw file is named:

```text
data/input/faostat-maize-yield-sample.csv
```

Do not open and resave the raw file in spreadsheet software — this can silently alter dates, encodings, numeric precision, delimiters, and quoting.

Create derived documentation and reports in separate directories; do not replace the source file.

---

## 1. Preserve and identify the raw input

Inspect file properties from the terminal:

```bash
ls -lh data/input/faostat-maize-yield-sample.csv
file data/input/faostat-maize-yield-sample.csv
sha256sum data/input/faostat-maize-yield-sample.csv
```

On systems without `sha256sum`, use an available SHA-256 tool and record which tool was used.

The checksum is a fingerprint of the file's bytes. If the file changes, its checksum should change. A matching checksum does not establish scientific correctness; it establishes that two byte sequences are the same.

Record:

- filename and relative path;
- file size;
- checksum;
- date received or accessed;
- who supplied or retrieved it.

---

## 2. Establish the dataset grain

Read the file without modifying it:

```r
maize_raw <- readr::read_csv(
  "data/input/faostat-maize-yield-sample.csv",
  show_col_types = FALSE
)

dplyr::glimpse(maize_raw)
names(maize_raw)
nrow(maize_raw)
```

Use provider documentation and the columns to complete this statement:

> One row represents ________________________________________________.

Propose a composite key. Adapt the names below to the actual extract:

```r
candidate_key <- c(
  "Area Code",
  "Year",
  "Item Code",
  "Element Code",
  "Unit"
)
```

Test it:

```r
duplicate_keys <- maize_raw |>
  dplyr::count(dplyr::across(dplyr::all_of(candidate_key))) |>
  dplyr::filter(n > 1)

duplicate_keys
```

If duplicates appear, do not remove them immediately. Ask:

- Is the proposed key incomplete?
- Is there another flag, status, method, or version variable?
- Does the provider allow multiple records for the same apparent unit?
- Is this a genuine source anomaly?

Document your conclusion.

---

## 3. Create a data dictionary

Review or create `metadata/faostat-data-dictionary.csv` with one row per
retained FAOSTAT source variable.

Include:

```text
variable
label
definition
type
unit
role
allowed_values
missing_values
source
notes
```

For code variables, point to the relevant code list. For `Value`, explain that its unit depends on the associated element/unit variables. For flags, record the provider's definition rather than guessing from the letter.

Review the dictionary using these questions:

- Would a learner understand the variable without opening the source portal?
- Is the difference between label and identifier clear?
- Are units explicit?
- Are special missing codes documented?
- Are quality flags explained?
- Does each definition describe the concept rather than repeat the name?

---

## 4. Record provenance

Create `metadata/provenance.yml`:

```yaml
artifact: data/input/faostat-maize-yield-sample.csv
provider: Food and Agriculture Organization of the United Nations
dataset: "record the exact dataset name"
release: "record if available"
accessed: "YYYY-MM-DD"
source: "record URL, endpoint, or supplied teaching snapshot"
license: "record the applicable terms"
retrieval: "describe the retrieval or delivery process"
checksum_sha256: "paste the checksum"
metadata_reference: "record the source documentation"
notes: "record known limitations or revisions"
```

Do not write secrets, access tokens, passwords, or private URLs into the record.

Commit the provenance record and dictionary when they contain no restricted information. They make future changes to data meaning and source history visible in Git.

---

## 5. Organize and govern the project artifacts

Confirm that each artifact has one clear role:

| Artifact | Role | Typical Git decision |
| --- | --- | --- |
| `data/input/faostat-maize-yield-sample.csv` | Unchanged teaching input | Decide from size, license, sensitivity, and reproducibility |
| `metadata/faostat-data-dictionary.csv` | Meaning of FAOSTAT source variables | Commit when it contains no restricted information |
| `metadata/provenance.yml` | Source and artifact history | Commit after removing credentials or private locations |
| `scripts/validate-data.R` | Repeatable expectations and checks | Commit |
| `reports/data-validation.html` | Rendered findings for review | Follow the project's output policy |

Check the source license before deciding whether the raw file or derived outputs may be redistributed, and record the required citation. Personal, confidential, or sensitive data should follow the project's access and retention rules instead of the general repository.

Inspect the version-control state:

```bash
git status --short
git check-ignore -v data/input/faostat-maize-yield-sample.csv
```

The second command reports the matching ignore rule, if any; no output does not mean committing the file is appropriate.

Document the storage decision in the project README so another contributor never has to infer whether a missing raw file must be downloaded, requested, mounted, or restored from an archive.

---

## 6. Validate structure

Create `scripts/validate-data.R`. Start by expressing expectations as code.

### Required columns

```r
required_columns <- c(
  "Area Code",
  "Area",
  "Year",
  "Item Code",
  "Item",
  "Element Code",
  "Element",
  "Unit",
  "Value"
)

missing_columns <- setdiff(required_columns, names(maize_raw))
stopifnot(length(missing_columns) == 0)
```

### Coverage

```r
range(maize_raw$Year, na.rm = TRUE)
dplyr::n_distinct(maize_raw$`Area Code`)
dplyr::count(maize_raw, Area, sort = TRUE)
```

Compare the results with the intended countries and years; do not hard-code an expectation until you can explain where it came from.

### Candidate-key uniqueness

```r
stopifnot(nrow(duplicate_keys) == 0)
```

Use a clear error or report when the expectation fails. A failed check is useful evidence, not an inconvenience to suppress.

### Allowed categories

```r
dplyr::count(maize_raw, Element, Unit, sort = TRUE)
dplyr::count(maize_raw, Item, sort = TRUE)
```

Compare observed categories with the data dictionary and source code lists.

---

## 7. Validate meaning

Structural validity does not establish that values are plausible or correctly interpreted.

### Missingness

```r
maize_raw |>
  dplyr::summarise(
    dplyr::across(
      dplyr::everything(),
      ~ sum(is.na(.x))
    )
  )
```

Ask whether missingness varies by country, year, element, or flag. A blank value is not automatically zero.

### Ranges

```r
maize_raw |>
  dplyr::group_by(Element, Unit) |>
  dplyr::summarise(
    minimum = min(Value, na.rm = TRUE),
    maximum = max(Value, na.rm = TRUE),
    .groups = "drop"
  )
```

Plausible ranges depend on element and unit. A single universal threshold is not appropriate for production, area, and yield.

### Flags

```r
if ("Flag" %in% names(maize_raw)) {
  print(dplyr::count(maize_raw, Flag, sort = TRUE))
}
```

Use provider metadata to interpret each flag. Decide whether it should be retained, summarized, or used to qualify later conclusions.

### Internal consistency

If the dataset contains production, harvested area, and yield, investigate whether their definitions imply an approximate relationship. Check units and provider methodology before calculating anything. Differences may reflect rounding, conversions, estimates, or definitions rather than errors.

---

## 8. Report rather than silently repair

Render `reports/data-validation.html` or a Markdown equivalent. Include:

1. source and checksum;
2. intended use;
3. stated grain and candidate key;
4. dimensions and coverage;
5. required-column and type checks;
6. duplicate-key results;
7. categories, units, and flags;
8. missingness and range summaries;
9. failed checks and anomalies;
10. limitations and decisions deferred to data preparation.

Classify findings:

- **Pass:** observed data match a justified expectation.
- **Warning:** data can continue through the workflow, but the issue needs attention.
- **Failure:** the input is not the expected dataset or cannot safely proceed.
- **Unknown:** additional provider or subject-matter information is required.

Do not turn every warning into an automatic deletion — preserve the evidence and document the reasoning needed for later preparation.

---

## Troubleshooting

### Problem 01: Column names differ from the example

Inspect `names(maize_raw)`, adapt the exercise to the actual schema, and record the real names in the dictionary — do not rename columns inside the raw file.

### Problem 02: The candidate key is not unique

Inspect several duplicated key combinations for omitted dimensions or statuses, and consult the metadata before changing the key or removing rows.

### Problem 03: Numbers were imported as text

Inspect the problematic values and import specification — likely causes are non-numeric symbols, decimal conventions, or provider-specific missing codes. Document the source representation; conversion belongs in a derived step.

### Problem 04: The checksum differs from another learner's

Compare file sizes, source versions, access dates, and retrieval methods — do not assume either file is authoritative until the origins are established.

### Problem 05: A validation check stops the script

Read the failed expectation and inspect the relevant records. Change the check only when the expectation was wrong and the revised expectation can be justified.

---

## Completion checklist

- [ ] The raw input remains unchanged.
- [ ] Its file size and checksum are recorded.
- [ ] The dataset grain is stated in one sentence.
- [ ] The candidate key is tested and duplicate results are explained.
- [ ] Every project variable is documented in a data dictionary.
- [ ] Source, access, release, license, and retrieval are recorded.
- [ ] Every artifact has a clear role and documented storage/version-control decision.
- [ ] Citation, redistribution, sensitivity, access, and retention conditions were considered.
- [ ] Structural and semantic checks run from a script.
- [ ] Failed checks and unknowns remain visible.
- [ ] A validation report summarizes evidence and limitations.
- [ ] Git status contains only the artifacts intended for version control.

---

## Reflect on the application

Answer these questions after completing the checklist:

1. Which concept from the Concepts page changed how you interpreted the raw file?
2. Which candidate-key variables were necessary, and what would go wrong if one were omitted?
3. Which validation result required scientific judgment rather than only a technical check?
4. Which issue, if any, should stop the data from moving to Data Preparation?
5. Which warning can move forward if it remains documented and visible?
6. Which artifacts should be committed to Git, and which should be ignored or stored elsewhere? Justify the decision.
7. What can a future learner reproduce from your package, and what still depends on the external provider?

The next topic, **Data Acquisition & Integration**, asks how to retrieve this and a complementary source reproducibly, align their identifiers and coverage, and audit the resulting integration.

---

## Further resources

Use these resources while implementing or reviewing a data-management workflow:

### Practical workflow and checks

- [Research Data Management Checklist — The Turing Way](https://book.the-turing-way.org/reproducible-research/rdm/rdm-checklist.html) summarizes practical actions for raw-data preservation, planning, and documentation.
- [Data Cleaning — The Turing Way](https://book.the-turing-way.org/reproducible-research/rdm/rdm-cleaning/) discusses raw-data backups and reproducible cleaning; use it as a bridge to Data Preparation, not a guide for this application.
- [Data management checklist — UK Data Service](https://ukdataservice.ac.uk/learning-hub/research-data-management/plan-to-share/checklist/) offers questions for reviewing documentation, storage, access, and sharing.

### Organization and documentation

- [Organising data — UK Data Service](https://ukdataservice.ac.uk/learning-hub/research-data-management/format-your-data/organising/) gives guidance on folder structures and meaningful file names.
- [Study-level documentation — UK Data Service](https://ukdataservice.ac.uk/learning-hub/research-data-management/document-your-data/study-level-documentation/) is a checklist for documenting purpose, methods, coverage, and licensing.
- [Data documentation: quantitative data — UK Data Service](https://ukdataservice.ac.uk/learning-hub/research-data-management/document-your-data/data-level/data-documentation-quantitative-data/) supports the dictionary task with guidance on variables, units, and missing values.

### Working with tabular data in R

- [Starting with data — Data Carpentry](https://lessons.datacarpentry.org/R-ecology-lesson/02-starting-with-data.html) introduces importing CSV files and inspecting data frames in R.
- [Data Analysis and Visualisation in R for Ecologists — Data Carpentry](https://lessons.datacarpentry.org/R-ecology-lesson/) provides a broader hands-on sequence for organizing and analyzing tabular data.
