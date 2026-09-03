# 4.3) Documentation tools for data management

---

- Source: [04_03_data_management_tools.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/04_03_data_management_tools.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/04_03_data_management_tools.md)
- Feedback: [Topic 04: Data Management](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/5)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Why use plain-text documentation?](#why-use-plain-text-documentation)
  - [Human-readable and machine-readable are not opposites](#human-readable-and-machine-readable-are-not-opposites)
- [Choose a format by information structure](#choose-a-format-by-information-structure)
  - [Use Markdown when](#use-markdown-when)
  - [Use YAML when](#use-yaml-when)
  - [Use CSV when](#use-csv-when)
  - [Decision table](#decision-table)
- [How the formats work together](#how-the-formats-work-together)
  - [Prefer one authoritative owner](#prefer-one-authoritative-owner)
- [SHA-256 checksums for file identity](#sha-256-checksums-for-file-identity)
  - [What is a checksum?](#what-is-a-checksum)
  - [What a matching checksum establishes](#what-a-matching-checksum-establishes)
  - [What a checksum does not establish](#what-a-checksum-does-not-establish)
  - [Integrity is not authenticity](#integrity-is-not-authenticity)
  - [Why use SHA-256 rather than SHA-1 or MD5?](#why-use-sha-256-rather-than-sha-1-or-md5)
- [Continue to the reference page](#continue-to-the-reference-page)

---

## Learning objectives

After completing this page, you should be able to:

- choose Markdown, YAML, or CSV according to the structure and intended use of documentation;
- explain how the three formats complement one another in a project;
- explain what a SHA-256 checksum can and cannot establish about a file.

The [reference page](04_05_data_management_tools_reference.md) that follows this one covers the concrete syntax, a full checksum workflow, validation checks, and a worked application to the maize-yield project.

---

## Why use plain-text documentation?

Markdown, YAML, and CSV are plain-text formats. Plain text is useful in a data science project because it can be:

- opened with many editors and operating systems;
- reviewed without specialized software;
- searched with command-line tools;
- processed by scripts;
- compared line by line with Git;
- included in code review; and
- preserved more easily than an undocumented binary format.

Plain text does not automatically make documentation correct or reproducible. A text file can still be ambiguous, malformed, incomplete, stale, or unsafe to share. The project must define what each file is for and how it is checked.

---

### Human-readable and machine-readable are not opposites

The three formats occupy different positions:

| Format | Main strength | Typical reader |
| --- | --- | --- |
| Markdown | Narrative explanation and readable structure | People first; rendering tools second |
| YAML | Nested structured records and configuration | People and programs |
| CSV | Repeated rectangular records | Programs and spreadsheet/table tools |

A useful data-management setup normally uses several formats rather than forcing every kind of information into one file.

---

## Choose a format by information structure

Begin with the structure of the information, not with a preferred file extension.

---

### Use Markdown when

- the order of explanation matters;
- the document contains sections, guidance, decisions, or interpretation;
- readers need examples, tables, lists, links, and code snippets;
- the content is primarily narrative; or
- the document should render as a readable web page or report.

Examples:

- a data-management implementation guide;
- a README;
- a validation-report narrative;
- instructions for updating a snapshot; and
- a description of limitations and responsible use.

---

### Use YAML when

- the information consists of named fields;
- values are nested or grouped;
- one record contains lists or sub-records;
- software must retrieve fields by name; or
- the file configures a tool or records structured provenance.

Examples:

- a provenance record;
- source metadata;
- a source register;
- a pipeline configuration; and
- a module or report configuration.

---

### Use CSV when

- every record has the same fields;
- the information is naturally rectangular;
- one row represents one variable, code, country, source, or validation check;
- table operations such as filtering and joining are useful; or
- learners should inspect the information with R or a spreadsheet.

Examples:

- a data dictionary;
- a code list;
- an identifier crosswalk;
- a validation-results table; and
- a register with one row per artifact when no nested fields are required.

---

### Decision table

| Requirement | Markdown | YAML | CSV |
| --- | ---: | ---: | ---: |
| Long narrative | Strong | Weak | Poor |
| Headings and explanatory flow | Strong | Poor | Poor |
| Named fields | Possible | Strong | Strong |
| Nested structures | Awkward | Strong | Poor |
| Repeated uniform records | Possible | Possible | Strong |
| Tables with many rows | Weak | Weak | Strong |
| Comments | Native prose | Supported | No standard comment syntax |
| Easy line-by-line Git review | Strong | Strong | Strong for stable row order |
| Direct use in R | Via a parser/rendering tool | Via a YAML parser | Via a table reader |
| Formal schema validation | Tool-dependent | Possible | Possible |

---

## How the formats work together

A project can separate responsibilities like this:

```text
docs/data-management.md
  Markdown narrative: policy, workflow, decisions, and limitations

metadata/source-metadata.yml
  YAML record: provider dataset, methods, scope, classifications, references

metadata/data-dictionary.csv
  CSV table: one row per project variable

metadata/flag-code-list.csv
  CSV table: one row per provider flag

metadata/provenance.yml
  YAML record: exact artifact, access, retrieval, checksum, and conditions
```

The files should link to one another rather than repeat all content:

```text
Markdown guide
  ├── points to source metadata
  ├── points to dictionary and code lists
  └── explains how provenance is updated

Provenance
  ├── identifies the artifact
  ├── points to source metadata
  └── records the dictionary relevant to that artifact
```

---

### Prefer one authoritative owner

If the licence is recorded in several files, decide which one owns the exact text and which files reference it. Otherwise, one copy may change while another remains stale.

Duplication is sometimes useful for readability, but duplicated fields require a consistency check or a maintenance rule.

---

## SHA-256 checksums for file identity

### What is a checksum?

A cryptographic hash function maps the bytes of a file to a fixed-length digest. SHA-256 produces a 256-bit digest, commonly displayed as 64 hexadecimal characters.

For example:

```text
fd2c78cae5a5cf2f82d6b6bdc2b3637ce03b597f74e561099f9666af449605be
```

The same bytes produce the same digest. A changed byte should produce a different digest with extremely high probability.

---

### What a matching checksum establishes

If a newly calculated digest matches a separately recorded expected digest, the file is byte-for-byte identical to the state represented by that expected digest.

This can help answer:

- Is this the fixed teaching snapshot?
- Did a configuration file change?
- Was a downloaded file transferred without byte-level alteration?
- Does a generated artifact match a reviewed release?
- Did line endings, encoding, whitespace, or final newline change?

---

### What a checksum does not establish

A matching checksum does **not** prove that the file is:

- correct;
- complete in a scientific sense;
- well documented;
- valid according to a schema;
- free of bias or measurement error;
- safe to execute;
- licensed for the intended use;
- authentic when the expected checksum came from an untrusted source; or
- unchanged in meaning when external definitions have changed.

Checksums establish file identity, not data quality or fitness for purpose.

---

### Integrity is not authenticity

Suppose an attacker can replace both:

```text
configuration.yml
configuration.yml.sha256
```

The new file can match the new checksum. The checksum alone does not identify who approved it.

For stronger authenticity, obtain the expected digest through a trusted channel or combine checksums with controls such as:

- reviewed Git history;
- protected branches;
- signed commits or tags;
- digital signatures;
- access controls; and
- published release manifests.

---

### Why use SHA-256 rather than SHA-1 or MD5?

SHA-256 is part of the SHA-2 family and is appropriate for routine file identity and integrity checks. MD5 and SHA-1 should not be selected for new security-sensitive integrity designs because of known collision weaknesses.

The algorithm must always be recorded with the digest. A hexadecimal string without an algorithm name is ambiguous.

---

## Continue to the reference page

The [tools reference page](04_05_data_management_tools_reference.md) covers concrete Markdown, YAML, and CSV syntax and guidelines, a step-by-step checksum workflow, validation checks per format, the application to the maize-yield project, common mistakes, and completion checklists. Return here whenever you need to decide which format to use; use the reference page when you need the exact syntax or workflow.
