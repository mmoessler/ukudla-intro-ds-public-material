# 4.5) Documentation tools for data management — reference

---

- Source: [04_05_data_management_tools_reference.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/04_05_data_management_tools_reference.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/04_05_data_management_tools_reference.md)
- Feedback: [Topic 04: Data Management](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/5)

---

This page is the detailed reference companion to
[Documentation tools for data management](04_03_data_management_tools.md). Read
that page first for the concepts — why plain text, how to choose Markdown,
YAML, or CSV, and what a checksum establishes. This page collects the
concrete syntax, a full checksum workflow, validation checks, a maize-yield
worked example with transfer guidance, common mistakes, and completion
checklists.

---

## Outline

- [Outline](#outline)
- [Markdown for explanations and guidance](#markdown-for-explanations-and-guidance)
  - [What Markdown represents](#what-markdown-represents)
  - [Basic structure](#basic-structure)
  - [Lists](#lists)
  - [Tables](#tables)
  - [Code and file names](#code-and-file-names)
  - [Links](#links)
  - [Markdown guidelines for data management](#markdown-guidelines-for-data-management)
  - [Markdown limitations](#markdown-limitations)
- [YAML for structured records and configuration](#yaml-for-structured-records-and-configuration)
  - [What YAML represents](#what-yaml-represents)
  - [Mapping](#mapping)
  - [Sequence](#sequence)
  - [Nested record](#nested-record)
  - [Literal and folded text](#literal-and-folded-text)
  - [Quote ambiguous values](#quote-ambiguous-values)
  - [Comments](#comments)
  - [YAML for provenance](#yaml-for-provenance)
  - [YAML for source metadata](#yaml-for-source-metadata)
  - [YAML guidelines for data management](#yaml-guidelines-for-data-management)
  - [YAML limitations](#yaml-limitations)
- [CSV for rectangular documentation](#csv-for-rectangular-documentation)
  - [What CSV represents](#what-csv-represents)
  - [Simple example](#simple-example)
  - [Quoting](#quoting)
  - [CSV for a data dictionary](#csv-for-a-data-dictionary)
  - [CSV for a code list](#csv-for-a-code-list)
  - [CSV for a crosswalk](#csv-for-a-crosswalk)
  - [CSV guidelines for data management](#csv-guidelines-for-data-management)
  - [CSV limitations](#csv-limitations)
- [A practical checksum workflow](#a-practical-checksum-workflow)
  - [1. Choose the artifact](#1-choose-the-artifact)
  - [2. Calculate SHA-256 on Linux](#2-calculate-sha-256-on-linux)
  - [3. Calculate SHA-256 on macOS](#3-calculate-sha-256-on-macos)
  - [4. Calculate SHA-256 with PowerShell](#4-calculate-sha-256-with-powershell)
  - [5. Calculate SHA-256 in R](#5-calculate-sha-256-in-r)
  - [6. Record the expected digest](#6-record-the-expected-digest)
  - [7. Use a checksum manifest](#7-use-a-checksum-manifest)
  - [8. Verify in R](#8-verify-in-r)
  - [9. Investigate a mismatch](#9-investigate-a-mismatch)
- [Validating documentation and configuration](#validating-documentation-and-configuration)
  - [Four different questions](#four-different-questions)
  - [Markdown checks](#markdown-checks)
  - [YAML checks](#yaml-checks)
  - [CSV checks](#csv-checks)
  - [Byte identity versus semantic equivalence](#byte-identity-versus-semantic-equivalence)
- [Worked example: the maize-yield project](#worked-example-the-maize-yield-project)
  - [Example workflow](#example-workflow)
  - [Appropriate checksum targets](#appropriate-checksum-targets)
  - [Suggested validation responsibilities](#suggested-validation-responsibilities)
- [Common mistakes](#common-mistakes)
  - [Choosing by file extension rather than structure](#choosing-by-file-extension-rather-than-structure)
  - [Treating Markdown as structured configuration](#treating-markdown-as-structured-configuration)
  - [Treating YAML as free-form prose](#treating-yaml-as-free-form-prose)
  - [Treating CSV as a spreadsheet layout](#treating-csv-as-a-spreadsheet-layout)
  - [Relying on automatic types](#relying-on-automatic-types)
  - [Accepting a parse as validation](#accepting-a-parse-as-validation)
  - [Updating a checksum without reviewing the change](#updating-a-checksum-without-reviewing-the-change)
  - [Storing the only expected digest with the untrusted artifact](#storing-the-only-expected-digest-with-the-untrusted-artifact)
  - [Hashing a logical value instead of file bytes](#hashing-a-logical-value-instead-of-file-bytes)
  - [Assuming a checksum proves correctness](#assuming-a-checksum-proves-correctness)
  - [Committing secrets in YAML](#committing-secrets-in-yaml)
- [Completion checklist](#completion-checklist)
  - [Format choice](#format-choice)
  - [Markdown](#markdown)
  - [YAML](#yaml)
  - [CSV](#csv)
  - [Checksums](#checksums)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
  - [File formats](#file-formats)
  - [Checksums](#checksums-1)
  - [Data-management context](#data-management-context)

---

## Markdown for explanations and guidance

### What Markdown represents

Markdown represents the structure of a document using plain-text conventions. It is suitable for prose, headings, lists, links, quotations, tables, and code examples.

Markdown implementations have dialects and extensions. A project should state which renderer or convention it expects. CommonMark provides a strongly specified core syntax; platforms may add tables, task lists, footnotes, or other features.

---

### Basic structure

````markdown
# Data-management implementation

This document explains how the project manages its teaching data.

## Raw-data policy

- Preserve the tracked teaching snapshot.
- Do not edit raw files manually.
- Ignore large provider downloads.

## Validate the snapshot

Run:

```bash
Rscript scripts/validate-data.R
```
````

Use heading levels hierarchically:

```text
# Document title
## Main section
### Subsection
```

Do not choose a heading level only for its visual size. Heading hierarchy communicates document structure to renderers, accessibility tools, and readers.

---

### Lists

Use bulleted lists when order does not matter:

```markdown
- provider
- dataset
- access date
- licence
```

Use numbered lists when sequence matters:

```markdown
1. Acquire the source file.
2. Record its checksum.
3. Validate its structure.
4. Review the results.
```

---

### Tables

Markdown tables work well for small comparison or decision tables:

```markdown
| Artifact | Role | Git policy |
| --- | --- | --- |
| Teaching snapshot | Fixed input | Track |
| Bulk download | External working file | Ignore |
```

Do not use a large Markdown table as a substitute for a machine-readable CSV. Wide tables become hard to edit, parse, and review.

---

### Code and file names

Use inline code for commands, field names, and repository-relative paths:

```markdown
Run `scripts/validate-data.R` and inspect the `status` column.
```

Use fenced code blocks for examples:

````markdown
```yaml
provider: FAO
dataset: Crops and Livestock Products
```
````

Specify a language after the opening fence when possible. This improves readability and syntax highlighting.

---

### Links

Prefer descriptive link text:

```markdown
[FAOSTAT Crops and Livestock Products](https://www.fao.org/faostat/en/#data/QCL)
```

Avoid:

```markdown
[click here](https://example.org)
```

For repository files, prefer relative links so that they work in different clones:

```markdown
`metadata/faostat-data-dictionary.csv`
```

---

### Markdown guidelines for data management

A data-management Markdown document should usually state:

- purpose and intended audience;
- scope and boundaries;
- the artifacts and scripts involved;
- the grain and role of important datasets;
- storage and version-control decisions;
- validation and maintenance procedures;
- licence, sensitivity, access, and sharing considerations;
- known limitations; and
- links to structured metadata, dictionaries, provenance, and code.

---

### Markdown limitations

Markdown is flexible, which means that software cannot reliably infer every important field from prose. For example, this sentence is readable:

> The snapshot was accessed on 3 August 2026.

But a program may have difficulty extracting the date consistently. If a script must use the value, store it in a structured record such as YAML and refer to that record from the narrative.

---

## YAML for structured records and configuration

### What YAML represents

YAML represents mappings, sequences, and scalar values using indentation and plain-text syntax.

The core structures are:

- **mapping**: named key–value pairs;
- **sequence**: an ordered list; and
- **scalar**: a string, number, Boolean, date-like value, or null value.

---

### Mapping

```yaml
provider: Food and Agriculture Organization of the United Nations
database: FAOSTAT
dataset: Crops and Livestock Products
```

Keys should have stable, documented names. Within one mapping, every key should be unique.

---

### Sequence

```yaml
elements:
  - Area harvested
  - Production
  - Yield
```

---

### Nested record

```yaml
coverage:
  countries: 9
  first_year: 1990
  last_year: 2022
```

Indent nested content consistently with spaces. Do not use tab characters for indentation.

---

### Literal and folded text

Use `|` when line breaks must be retained:

```yaml
note: |
  Preserve the source file unchanged.
  Record transformations in derived scripts.
```

Use `>-` for a readable multi-line paragraph that should be loaded as one folded string without a final newline:

```yaml
known_limitations: >-
  Historical observations may be revised, and reporting methods can differ
  among countries.
```

---

### Quote ambiguous values

Some YAML parsers interpret unquoted values as numbers, dates, Booleans, or nulls. Quote values when their exact textual representation matters:

```yaml
accessed: "2026-08-03"
release: "061"
country_code: "008"
answer: "yes"
```

Quoting is especially important for:

- identifiers with leading zeros;
- version numbers;
- date-like strings;
- values such as `yes`, `no`, `on`, `off`, `true`, `false`, `null`, or `~` when they are intended as text; and
- strings containing `: ` or ` #`.

Parser behavior can differ between YAML versions and libraries. Validate with the same parser used by the project.

---

### Comments

```yaml
release: "061" # MODIS product collection
```

Comments help maintainers, but programs normally discard them after parsing. Do not store information required by software only in a comment.

---

### YAML for provenance

YAML is suitable when one artifact has nested provenance fields:

```yaml
artifact: data/input/faostat-maize-yield-sample.csv
provider: Food and Agriculture Organization of the United Nations
dataset: Crops and Livestock Products
accessed: "2026-08-03"
retrieval:
  script: scripts/acquire-faostat-data.R
  method: Bulk ZIP download
selection:
  countries: 9
  years: 1990-2022
  item: Maize (corn)
checksum:
  algorithm: SHA-256
  value: fd2c78cae5a5cf2f82d6b6bdc2b3637ce03b597f74e561099f9666af449605be
```

---

### YAML for source metadata

```yaml
provider: Food and Agriculture Organization of the United Nations
database: FAOSTAT
dataset: Crops and Livestock Products
dataset_code: QCL
frequency: Annual
classifications:
  item: Agricultural commodity
  element: Statistical measure
quality_flags:
  local_code_list: metadata/faostat-flag-code-list.csv
```

---

### YAML guidelines for data management

- Use one documented meaning for every key.
- Prefer descriptive keys such as `checksum_sha256` over `hash`.
- Keep key naming consistent, for example `snake_case` throughout.
- State whether a field is required, optional, or conditionally required.
- Use a list for repeated values rather than a comma-separated string.
- Keep paths repository-relative when the artifact belongs to the project.
- Do not place passwords, tokens, private keys, or restricted URLs in tracked YAML.
- Parse the file in an automated check; visual inspection is insufficient.
- Consider a schema when several records must follow the same structure.

---

### YAML limitations

YAML is sensitive to indentation and type interpretation. It is not ideal for large tables with hundreds of similar rows. Duplicate keys may be accepted, rejected, or silently overwritten depending on the parser. A file can parse successfully while still lacking required fields or containing scientifically invalid values.

---

## CSV for rectangular documentation

### What CSV represents

CSV represents a table as delimited text. In the usual form:

- the first row contains column names;
- each later row represents one record;
- commas separate fields; and
- double quotes protect fields containing commas, quotes, or line breaks.

---

### Simple example

```csv
flag,description,source
A,Official figure,FAOSTAT
E,Estimated value,FAOSTAT
```

---

### Quoting

A comma inside a field requires quotes:

```csv
variable,definition,notes
value,Reported numeric quantity,"Interpret with element, unit, and flag."
```

A literal double quote inside a quoted field is represented by two double quotes:

```csv
term,definition
raw,"The project's ""raw"" artifact role"
```

---

### CSV for a data dictionary

```csv
variable,label,definition,type,unit,role,allowed_values,missing_values
area,Area,Reporting geographic area,character,,label,Project countries,blank not expected
year,Year,Calendar year,integer,year,time,1990-2022,blank not expected
value,Value,Reported numeric quantity,double,varies,measure,non-negative,blank means missing
```

One row represents one variable. The header defines the documentation schema.

---

### CSV for a code list

```csv
code,label,definition
A,Official figure,Value reported as an official figure
E,Estimated value,Value estimated by the provider
```

One row represents one allowed code.

---

### CSV for a crosswalk

```csv
project_country_id,faostat_label,other_source_code,note
ZAF,South Africa,ZAF,Reviewed mapping
ZMB,Zambia,ZMB,Reviewed mapping
```

One row represents one mapping. State the expected key and valid direction of the mapping before using it in a join.

### CSV guidelines for data management

- Use a single header row with unique, stable names.
- Use UTF-8 encoding unless the project explicitly requires another encoding.
- Define the delimiter, decimal mark, quote character, and missing-value representation.
- Use one record type and one grain per table.
- Do not use formatting, color, comments, or merged cells to communicate meaning.
- Do not place titles or explanatory paragraphs above the header.
- Quote fields according to the writer/parser rather than inserting quotes manually.
- Preserve identifiers such as `008` as text when leading zeros matter.
- Use ISO-style dates such as `2026-08-03` when a date field is required.
- Keep row ordering stable when it has no semantic meaning but helps Git review.
- Validate required columns, types, keys, allowed values, and missingness.
- Store dataset-level explanation in Markdown or YAML rather than forcing it into every row.

---

### CSV limitations

CSV does not carry a standard data schema. It normally does not preserve:

- data types;
- units;
- key constraints;
- categorical definitions;
- missing-value meaning;
- relationships to other tables; or
- dataset-level provenance.

These must be documented and validated separately. Spreadsheet software may also convert identifiers, dates, or large numbers automatically. Inspect the file as text and read it with explicit types when those conversions matter.

CSV conventions vary. RFC 4180 documents a widely used common format, but real files may use semicolons, tabs, different line endings, or locale-specific decimal marks. Record the actual convention rather than assuming it.

---

## A practical checksum workflow

### 1. Choose the artifact

Record a repository-relative path:

```text
metadata/provenance.yml
```

Be clear whether the checksum applies to:

- the original provider response;
- a normalized teaching snapshot;
- a configuration file;
- a documentation file; or
- a generated release artifact.

Each artifact has a different role and may require a different expected digest.

---

### 2. Calculate SHA-256 on Linux

```bash
sha256sum metadata/provenance.yml
```

Typical output:

```text
<64-character-digest>  metadata/provenance.yml
```

---

### 3. Calculate SHA-256 on macOS

```bash
shasum -a 256 metadata/provenance.yml
```

---

### 4. Calculate SHA-256 with PowerShell

```powershell
Get-FileHash metadata/provenance.yml -Algorithm SHA256
```

---

### 5. Calculate SHA-256 in R

With the `digest` package:

```r
digest::digest(
  "metadata/provenance.yml",
  algo = "sha256",
  serialize = FALSE,
  file = TRUE
)
```

The `serialize = FALSE` and `file = TRUE` arguments ensure that the file bytes, rather than an R serialization of a character value, are hashed.

---

### 6. Record the expected digest

An artifact provenance record can contain:

```yaml
artifact: metadata/source-metadata.yml
checksum_algorithm: SHA-256
checksum_sha256: "<64-character-digest>"
```

Avoid recording a file's checksum only inside that same file when the goal is to verify the complete file. Adding the digest changes the file and therefore changes its digest. Store the expected value in:

- a separate provenance record;
- a checksum manifest;
- a signed release record; or
- validation code tied to a reviewed release.

---

### 7. Use a checksum manifest

Create a manifest for several stable files:

```bash
sha256sum \
  metadata/source-metadata.yml \
  metadata/faostat-data-dictionary.csv \
  metadata/faostat-flag-code-list.csv \
  > metadata/checksums.sha256
```

Verify it on Linux:

```bash
sha256sum --check metadata/checksums.sha256
```

On macOS:

```bash
shasum -a 256 --check metadata/checksums.sha256
```

Review the manifest diff before committing it. Do not regenerate expected digests automatically after a failed verification: first determine why the files changed and whether the change is authorized.

---

### 8. Verify in R

```r
expected <- "<recorded-sha256>"

observed <- digest::digest(
  "metadata/source-metadata.yml",
  algo = "sha256",
  serialize = FALSE,
  file = TRUE
)

if (!identical(observed, expected)) {
  stop("Source metadata do not match the reviewed state.")
}
```

---

### 9. Investigate a mismatch

Do not immediately replace the expected checksum. Check:

```bash
git status --short
git diff -- metadata/provenance.yml
git log --oneline -- metadata/provenance.yml
```

Ask:

- Was the file intentionally edited?
- Did a formatter reorder fields or change whitespace?
- Did spreadsheet software change encoding, quoting, dates, or line endings?
- Was the artifact regenerated from a new source release?
- Is the expected checksum for a different artifact or version?
- Has an unexpected or unauthorized change occurred?

Only approve and record the new digest after reviewing the change.

---

## Validating documentation and configuration

A checksum should be one layer in a broader validation strategy.

---

### Four different questions

| Check | Question answered |
| --- | --- |
| Parse check | Is the file syntactically readable in its format? |
| Schema/content check | Are required fields, types, keys, and allowed values present? |
| Checksum check | Is the file byte-identical to the reviewed state? |
| Human review | Is the meaning correct, current, justified, and fit for purpose? |

All four can be necessary.

---

### Markdown checks

Possible checks include:

- the file is valid UTF-8;
- heading hierarchy is sensible;
- internal and external links resolve;
- fenced code blocks are closed;
- required sections exist;
- commands and examples have been tested; and
- a Markdown linter or renderer accepts the project dialect.

A Markdown checksum detects any byte change, including harmless line wrapping. Use it only when exact byte identity matters, for example for a published release document. Git review and content checks are usually more useful during normal editing.

---

### YAML checks

At minimum, parse the file with the project parser.

In R:

```r
record <- yaml::read_yaml("metadata/provenance.yml")
```

Then check required keys:

```r
required <- c(
  "artifact", "provider", "dataset", "accessed",
  "retrieval", "checksum_sha256", "license"
)

missing <- setdiff(required, names(record))

if (length(missing) > 0) {
  stop("Missing provenance field(s): ", paste(missing, collapse = ", "))
}
```

Also validate:

- unique keys;
- expected data types;
- allowed values;
- date and identifier formats;
- referenced paths;
- URL or identifier structure where appropriate; and
- consistency with related records.

---

### CSV checks

Read with an explicit table parser:

```r
dictionary <- readr::read_csv(
  "metadata/faostat-data-dictionary.csv",
  show_col_types = FALSE
)
```

Check the schema and key:

```r
required_columns <- c(
  "variable", "definition", "type", "unit", "role"
)

stopifnot(all(required_columns %in% names(dictionary)))
stopifnot(!anyDuplicated(dictionary$variable))
stopifnot(!any(is.na(dictionary$variable)))
stopifnot(!any(is.na(dictionary$definition)))
```

Also inspect:

- unexpected extra columns;
- blank headers;
- encoding and delimiter;
- inferred versus intended types;
- duplicated rows or keys;
- missing required fields;
- allowed categories;
- references to code lists; and
- consistency with the actual data file.

---

### Byte identity versus semantic equivalence

These two YAML documents can represent equivalent mappings:

```yaml
provider: FAO
dataset: FAOSTAT
```

```yaml
dataset: FAOSTAT
provider: FAO
```

Their raw bytes and SHA-256 digests differ. Similarly, CSV files can differ in line endings or quotation while a parser returns the same table.

Therefore:

- use a checksum when exact file identity matters;
- use parsing and content checks when meaning matters;
- use both when a reviewed release requires exact identity and valid content.

Canonicalizing a file before hashing is possible, but it creates another specification: the project must define the parser, type rules, sorting, encoding, whitespace, and serialization method. For introductory projects, hashing the original bytes and validating content separately is clearer.

---

## Worked example: the maize-yield project

The formats and checksum methods above are general. The following section
shows one concrete combination for secondary data. Laboratory projects can
use the same formats for sample and assay documentation, field experiments for
treatment and randomization records, and observational studies for sampling,
site, visit, and measurement metadata.

The project can use the formats as follows:

| Artifact | Format | Reason |
| --- | --- | --- |
| `README.md` | Markdown | Project purpose, workflow, commands, and interpretation |
| `docs/data-management.md` | Markdown | Repository-specific implementation guidance |
| `metadata/source-metadata.yml` | YAML | Nested provider, methodology, scope, classification, and quality record |
| `metadata/provenance.yml` | YAML | Structured identity and history of the fixed teaching artifact |
| `metadata/faostat-data-dictionary.csv` | CSV | One row per field in the FAOSTAT teaching extract |
| `metadata/faostat-flag-code-list.csv` | CSV | One row per FAOSTAT quality flag |
| `metadata/checksums.sha256` | Checksum manifest | Expected byte identity of selected stable artifacts, if adopted |

---

### Example workflow

```text
Read source metadata
        ↓
Understand provider concepts, methods, units, flags, and limitations
        ↓
Inspect the project data dictionary and code lists
        ↓
Confirm the exact teaching artifact through provenance and checksum
        ↓
Parse and validate YAML and CSV structure
        ↓
Review meaning, licence, sensitivity, and fitness for purpose
```

---

### Appropriate checksum targets

Checksums are most useful for:

- a fixed raw teaching snapshot;
- an instructor-approved metadata release;
- a configuration file deployed to a service;
- a checksum manifest distributed with course materials; and
- an exported or archived release artifact.

Checksums are less useful as a replacement for Git review on actively edited Markdown. Every spelling correction changes the digest, but the digest does not explain the change.

---

### Suggested validation responsibilities

| File | Automated checks | Human review |
| --- | --- | --- |
| Markdown implementation guide | Required headings, links, render/lint | Accuracy, clarity, current workflow |
| YAML source metadata | Parse, required keys, allowed structure | Provider meaning, methods, references |
| YAML provenance | Parse, required keys, checksum format, referenced artifact | Retrieval history, licence, sharing decision |
| CSV dictionary | Header, unique variable key, non-empty definitions, allowed roles | Field meaning, units, missingness |
| CSV code list | Header, unique code key, referenced codes covered | Provider definitions and qualifications |
| Teaching data snapshot | Checksum, schema, grain, key, ranges | Fitness for purpose and limitations |

---

## Common mistakes

### Choosing by file extension rather than structure

A large YAML list of hundreds of identical variable records is harder to work with than CSV. A CSV containing paragraphs and nested lists is harder to understand than Markdown or YAML.

---

### Treating Markdown as structured configuration

A program should not have to search prose for the access date or checksum. Store required fields in a structured record and explain them in Markdown.

---

### Treating YAML as free-form prose

Very long narrative values make structured fields hard to review. Keep concise summaries in YAML and link to Markdown or authoritative documentation.

---

### Treating CSV as a spreadsheet layout

Do not add blank rows, merged headings, colors, formulas, or multiple tables to a CSV. Those features are either not represented or are interpreted inconsistently.

---

### Relying on automatic types

Identifiers can lose leading zeros, and date-like or Boolean-like YAML values can be converted unexpectedly. Define types and quote ambiguous YAML scalars.

---

### Accepting a parse as validation

A YAML file containing `provider: unknown` can parse successfully. A CSV with duplicate variable names can also parse successfully. Syntax is only the first validation layer.

---

### Updating a checksum without reviewing the change

A mismatch is a signal to investigate. Automatically replacing the expected digest removes the control the checksum was meant to provide.

---

### Storing the only expected digest with the untrusted artifact

If both can be replaced together, a matching digest does not establish authenticity. Use a trusted manifest, reviewed Git history, signature, or protected release channel.

---

### Hashing a logical value instead of file bytes

Programming libraries may hash a serialized object by default. When validating a file, confirm that the function reads and hashes the file's bytes.

---

### Assuming a checksum proves correctness

A perfectly preserved file can contain incorrect values or inappropriate methodology. Combine identity checks with metadata, dictionaries, provenance, validation, and scientific review.

---

### Committing secrets in YAML

Configuration syntax makes credentials look like ordinary fields. Keep secrets outside tracked files, supply them through an approved secret mechanism, and rotate any credential exposed in Git history.

---

## Completion checklist

### Format choice

- [ ] Narrative guidance is stored in Markdown.
- [ ] Nested records and configuration are stored in YAML when appropriate.
- [ ] Repeated rectangular records are stored in CSV.
- [ ] Each file has one documented purpose and grain.

---

### Markdown

- [ ] The heading hierarchy is logical.
- [ ] Lists, tables, links, and code blocks are used consistently.
- [ ] Repository links are relative where appropriate.
- [ ] Commands and examples have been checked.
- [ ] Structured values required by software are not hidden only in prose.

---

### YAML

- [ ] Indentation uses spaces rather than tabs.
- [ ] Keys are unique, stable, and consistently named.
- [ ] Ambiguous strings and identifiers are quoted.
- [ ] Repeated values use sequences rather than comma-separated strings.
- [ ] The file parses with the project's YAML parser.
- [ ] Required keys, types, values, and references are validated.
- [ ] No secret or restricted value is committed.

---

### CSV

- [ ] There is exactly one header row with unique field names.
- [ ] Every row has the same grain and record type.
- [ ] Encoding, delimiter, quotation, decimal, and missing-value conventions are known.
- [ ] Candidate keys are tested for uniqueness.
- [ ] Required fields and allowed values are validated.
- [ ] Dataset-level narrative and provenance are stored elsewhere.

---

### Checksums

- [ ] SHA-256 is named explicitly as the algorithm.
- [ ] The intended artifact path and version are clear.
- [ ] The expected digest is stored through a trusted, reviewed mechanism.
- [ ] Verification hashes file bytes rather than a serialized program object.
- [ ] A mismatch stops the workflow and triggers investigation.
- [ ] Checksums are supplemented by syntax, schema, content, and human review.
- [ ] New digests are recorded only after intentional changes are reviewed.

---

## Check your understanding

1. Why is Markdown usually better than YAML for a long implementation guide?
2. Why is CSV usually better than Markdown for a 200-variable data dictionary?
3. Give an example of information that belongs in YAML provenance and not only in Markdown prose.
4. Why should an identifier such as `008` be treated carefully in YAML and spreadsheet software?
5. What does a matching SHA-256 checksum establish?
6. Name four important properties that a checksum does not establish.
7. Why should a failed checksum not be “fixed” immediately by generating a new expected value?
8. How can two semantically equivalent YAML documents have different checksums?
9. Why is storing a checksum beside a file insufficient against an attacker who can replace both?
10. Which checks would you apply to `metadata/faostat-data-dictionary.csv` in addition to a checksum?

---

## Further resources

### File formats

- [CommonMark specification](https://spec.commonmark.org/spec) defines a
  consistent core Markdown syntax.
- [CommonMark quick reference](https://commonmark.org/help/) provides a compact
  introduction to Markdown elements.
- [YAML 1.2.2 specification](https://yaml.org/spec/1.2.2/) defines YAML's data
  model and syntax.
- [RFC 4180](https://www.rfc-editor.org/info/rfc4180/) documents a common CSV
  format and the `text/csv` media type. Real-world CSV files can still follow
  different conventions, so projects must record what they use.

---

### Checksums

- [NIST FIPS 180-4: Secure Hash Standard](https://csrc.nist.gov/pubs/fips/180-4/upd1/final)
  specifies SHA-256 and related secure hash algorithms.
- The R [`digest`](https://cran.r-project.org/package=digest) package can
  calculate file digests programmatically.

---

### Data-management context

- [The Turing Way: Research Data Management](https://book.the-turing-way.org/reproducible-research/rdm/)
  connects documentation, storage, sharing, preservation, and reproducibility.
- [Metadata, data dictionaries, and provenance](../../docs/topics/dm-metadata-dictionary-provenance.md)
  provides a detailed conceptual comparison and applies it to the maize-yield
  project.
