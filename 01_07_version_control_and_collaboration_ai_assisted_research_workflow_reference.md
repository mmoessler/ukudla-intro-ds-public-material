# 1.7) AI-assisted research workflow reference

---

- Source: [01_07_version_control_and_collaboration_ai_assisted_research_workflow_reference.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/01_07_version_control_and_collaboration_ai_assisted_research_workflow_reference.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/01_07_version_control_and_collaboration_ai_assisted_research_workflow_reference.md)
- Feedback: [Topic 01: Version Control and Collaboration](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/2)

---

This page provides the full procedural detail, checklists, and worked example
for the [reproducible AI-assisted research workflow](01_06_version_control_and_collaboration_ai_assisted_research_workflow.md).
Read that page first for the concepts, the condensed review–verify–commit
loop, and prompting guidance.

---

## Outline

- [Outline](#outline)
- [The evidence to preserve](#the-evidence-to-preserve)
  - [Do not turn Git into a chat archive](#do-not-turn-git-into-a-chat-archive)
  - [Suggested AI-use record](#suggested-ai-use-record)
- [1. Begin from a known repository state](#1-begin-from-a-known-repository-state)
  - [Synchronize deliberately](#synchronize-deliberately)
  - [Record the starting point when it matters](#record-the-starting-point-when-it-matters)
- [2. Define one bounded task](#2-define-one-bounded-task)
  - [Task brief template](#task-brief-template)
  - [Why bounded tasks improve reproducibility](#why-bounded-tasks-improve-reproducibility)
- [3. Choose a safe working context](#3-choose-a-safe-working-context)
  - [Branch naming examples](#branch-naming-examples)
  - [Separate exploratory and accepted work](#separate-exploratory-and-accepted-work)
- [4. Give the agent sufficient context and boundaries](#4-give-the-agent-sufficient-context-and-boundaries)
  - [Permission boundaries](#permission-boundaries)
  - [Repository instructions](#repository-instructions)
- [5. Inspect every change](#5-inspect-every-change)
  - [Review untracked files](#review-untracked-files)
  - [Review history-sensitive files carefully](#review-history-sensitive-files-carefully)
  - [Ask the agent to explain, then verify independently](#ask-the-agent-to-explain-then-verify-independently)
- [6. Verify behavior and research meaning](#6-verify-behavior-and-research-meaning)
  - [Software verification](#software-verification)
  - [Data verification](#data-verification)
  - [Scientific verification](#scientific-verification)
  - [Record what was not verified](#record-what-was-not-verified)
- [7. Record material decisions](#7-record-material-decisions)
  - [Avoid unsupported AI-generated citations](#avoid-unsupported-ai-generated-citations)
- [8. Commit one reviewed outcome](#8-commit-one-reviewed-outcome)
  - [Commit content](#commit-content)
  - [Commit message](#commit-message)
  - [A commit is not a certificate of correctness](#a-commit-is-not-a-certificate-of-correctness)
- [9. Integrate and publish deliberately](#9-integrate-and-publish-deliberately)
  - [Before merging](#before-merging)
  - [Tag research milestones](#tag-research-milestones)
- [Handling data, secrets, and external services](#handling-data-secrets-and-external-services)
  - [Classify information before sharing it](#classify-information-before-sharing-it)
  - [Never include secrets in prompts or tracked files](#never-include-secrets-in-prompts-or-tracked-files)
  - [Review agent tool use](#review-agent-tool-use)
- [Research integrity and disclosure](#research-integrity-and-disclosure)
  - [Verify facts and sources](#verify-facts-and-sources)
  - [Preserve intellectual contribution accurately](#preserve-intellectual-contribution-accurately)
  - [Follow applicable disclosure policy](#follow-applicable-disclosure-policy)
  - [Keep methods and writing distinct](#keep-methods-and-writing-distinct)
- [What cannot be reproduced exactly](#what-cannot-be-reproduced-exactly)
  - [Model and service changes](#model-and-service-changes)
  - [Nondeterminism](#nondeterminism)
  - [Hidden context](#hidden-context)
  - [External state](#external-state)
  - [Practical response](#practical-response)
- [Failure and recovery](#failure-and-recovery)
  - [The agent changed unrelated files](#the-agent-changed-unrelated-files)
  - [The proposed approach is wrong](#the-proposed-approach-is-wrong)
  - [Tests pass but results changed unexpectedly](#tests-pass-but-results-changed-unexpectedly)
  - [A citation cannot be verified](#a-citation-cannot-be-verified)
  - [A sensitive value was exposed](#a-sensitive-value-was-exposed)
  - [The session context is lost](#the-session-context-is-lost)
- [Application to the maize-yield project](#application-to-the-maize-yield-project)
  - [Scenario](#scenario)
  - [Starting checks](#starting-checks)
  - [Invariants](#invariants)
  - [Review](#review)
  - [Verification](#verification)
  - [Commit](#commit)
- [Practice exercise](#practice-exercise)
  - [Reflection](#reflection)
- [Completion checklist](#completion-checklist)
  - [Before AI assistance](#before-ai-assistance)
  - [During AI assistance](#during-ai-assistance)
  - [Before committing](#before-committing)
  - [Before publishing or merging](#before-publishing-or-merging)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
  - [Version control](#version-control)
  - [Responsible AI and research practice](#responsible-ai-and-research-practice)
  - [Reproducibility and documentation](#reproducibility-and-documentation)

---

## The evidence to preserve

Not every AI interaction needs to be committed. Preserve information in proportion to its importance for understanding or reproducing the research.

| Evidence | Usually preserve? | Suitable location |
| --- | --- | --- |
| Accepted source-code change | Yes | Git commit |
| Test and validation code | Yes | Repository |
| Environment definitions | Yes | Lockfile, container, or configuration |
| Data provenance and checksums | Yes | Metadata/provenance records |
| Research decision affected by AI | Yes | Commit body, issue, decision record, or methods note |
| Verification commands and results | Yes when material | Test configuration, report, log summary, or commit body |
| Tool/model/interface and date | When relevant to disclosure or reproduction | Methods note or AI-use record |
| Full conversation transcript | Sometimes | Approved research record outside Git when needed |
| Routine brainstorming | Usually no | Temporary notes, if useful |
| Secrets or confidential prompts | Never in Git | Approved secure system only |
| Unaccepted AI output | Usually no | Discard or retain only when needed for an audit |

---

### Do not turn Git into a chat archive

Committing every prompt and response can:

- expose sensitive data;
- preserve copyrighted or restricted material unnecessarily;
- overwhelm the project history;
- obscure the decisions that actually mattered; and
- create records that cannot be shared with the repository.

For a methodologically important interaction, record a concise, structured summary unless a complete transcript is required by the study protocol, institution, funder, journal, or audit process.

---

### Suggested AI-use record

For a material contribution, a short record might contain:

```markdown
## AI assistance record

- Date: 2026-08-06
- Tool/interface: coding agent in the project workspace
- Task: add validation for the teaching-data candidate key
- Repository starting commit: abc1234
- Human-supplied constraints: preserve raw data; modify validation only
- Accepted contribution: proposed duplicate-key check and documentation
- Human verification: reviewed diff; ran validation; inspected duplicate count
- Human decision: accepted check; rewrote interpretation paragraph
- Resulting commit: def5678
- Limitations: exact model version and generation cannot be replayed locally
```

Do not claim that this record makes the model response deterministic. It makes the accepted research decision easier to audit.

---

## 1. Begin from a known repository state

Move to the repository root and inspect it:

```bash
pwd
git status
git branch --show-current
git remote -v
git log --oneline --decorate -5
```

Confirm:

- this is the intended repository;
- the current branch is understood;
- existing changes belong to you or are otherwise accounted for;
- remotes point to the expected locations; and
- the starting commit is identifiable.

Do not ask an agent to "clean up everything" in a working tree that already contains unexplained changes. The agent may overwrite, combine, or misclassify someone else's work.

---

### Synchronize deliberately

If the project workflow requires an up-to-date base and the working tree is clean:

```bash
git pull --no-rebase origin main
```

Use the actual branch and remote defined by the project. Inspect fetched or merged changes before continuing.

---

### Record the starting point when it matters

```bash
git rev-parse HEAD
```

The commit ID is useful when an AI task spans several revisions or when a methods record needs to identify the exact input state.

---

## 2. Define one bounded task

A useful task describes an observable outcome.

Weak request:

> Improve the project.

More reviewable request:

> Add a validation check that stops when the teaching dataset contains a
> duplicated `area + item + element + year + unit` key. Update the validation
> report to explain the check. Do not change the raw data or analysis model.

---

### Task brief template

```text
Outcome:
Files or components in scope:
Files or components out of scope:
Starting evidence:
Scientific assumptions that must not change:
Required verification:
Expected documentation:
Actions requiring approval:
```

Example:

```text
Outcome: Document and test the fixed teaching-sample checksum.
In scope: metadata/provenance.yml, scripts/validate-data.R.
Out of scope: raw data, modeling scripts, package versions.
Starting evidence: checksum supplied with the reviewed snapshot.
Scientific assumptions: validation must not modify the input.
Required verification: matching checksum passes; modified copy fails.
Expected documentation: explain identity versus data quality.
Actions requiring approval: network access and dependency installation.
```

---

### Why bounded tasks improve reproducibility

They make it easier to:

- detect unrelated edits;
- define verification before implementation;
- understand a diff;
- reject an unsuitable approach;
- create a coherent commit; and
- explain the decision later.

---

## 3. Choose a safe working context

Use a branch for a feature, experiment, or uncertain change:

```bash
git switch -c feature/validate-teaching-data
```

A branch isolates history; it does not create a security boundary. An agent with filesystem or network access may still affect files, services, or data outside the branch.

---

### Branch naming examples

```text
feature/add-data-validation
docs/explain-provenance
experiment/compare-yield-models
bugfix/correct-unit-conversion
```

Choose a name that describes the purpose rather than the tool:

```text
feature/add-data-validation   preferred
ai-work                       too vague
```

---

### Separate exploratory and accepted work

Exploration can include failed approaches. The accepted branch history should still make clear which change was reviewed and why. Depending on project policy, retain experimental commits, squash them during review, or summarize the exploration in an issue or decision record.

Do not rewrite shared history without coordinating with collaborators.

---

## 4. Give the agent sufficient context and boundaries

An agent needs enough context to follow the project, but it should not receive unnecessary sensitive information or authority.

Provide:

- the research or software objective;
- relevant repository paths;
- coding and documentation conventions;
- data grain, units, and key assumptions;
- tests or expected behavior;
- files that must remain unchanged;
- whether network access is permitted;
- whether installing dependencies is permitted; and
- which actions require confirmation.

---

### Permission boundaries

Distinguish among:

| Action | Example | Typical treatment |
| --- | --- | --- |
| Read-only inspection | Read files, show status, parse configuration | Usually low risk within project scope |
| Reversible project edit | Modify one script on a branch | Review diff before acceptance |
| Environment mutation | Install packages, update lockfiles | Require explicit scope and review |
| External state change | Push, open issue, send message, start cloud job | Require clear authorization |
| Destructive action | Delete data, rewrite history, overwrite release | Require explicit approval and verified target |

Do not broaden authority merely because the agent proposes a convenient next step.

---

### Repository instructions

If a project has contributor instructions, style guides, tests, or data policies, point the agent to them. A tool cannot follow a convention it has not seen.

---

## 5. Inspect every change

After the agent works, inspect the repository yourself:

```bash
git status --short
git diff
```

Check:

- Are only intended files changed?
- Is every new file expected?
- Were data, credentials, generated outputs, or lockfiles added accidentally?
- Does the implementation match the task rather than merely compile?
- Are units, identifiers, joins, missing values, and boundary cases handled?
- Were comments and documentation updated?
- Did the change weaken validation or silently suppress errors?
- Is the implementation unnecessarily complex?

---

### Review untracked files

`git diff` does not display the content of untracked files. Use `git status` to find them and open each intended file before staging it.

---

### Review history-sensitive files carefully

Pay particular attention to:

- environment lockfiles;
- dependency manifests;
- `.gitignore`;
- CI/CD configuration;
- database migrations;
- data snapshots;
- submodule pointers;
- credentials and `.env` files; and
- generated reports presented as research results.

---

### Ask the agent to explain, then verify independently

Useful questions include:

- Which assumptions does this implementation make?
- Which files changed and why?
- What failure cases remain?
- Which verification was run?
- What was not tested?

An explanation helps review but is not evidence by itself. Compare it with the diff, commands, and results.

---

## 6. Verify behavior and research meaning

Verification should be proportional to the risk of the change.

---

### Software verification

Possible checks include:

- parse or syntax checks;
- unit tests;
- integration tests;
- schema and configuration validation;
- a clean render or build;
- reproducibility from an empty output directory;
- comparison with a known result; and
- inspection of logs and warnings.

---

### Data verification

Check relevant properties such as:

- input identity and provenance;
- row counts before and after transformation;
- grain and candidate-key uniqueness;
- units and conversions;
- missingness;
- allowed classifications and flags;
- unmatched and multiplied join keys;
- temporal and geographic coverage; and
- unexpected changes in summaries or distributions.

---

### Scientific verification

Tests can pass while the method is inappropriate. Ask:

- Does the method answer the stated research question?
- Are causal, predictive, and descriptive claims distinguished?
- Are assumptions defensible?
- Is leakage introduced between training and evaluation data?
- Are uncertainty and limitations reported?
- Are output interpretations supported by the data and model?
- Did AI-generated text overstate the evidence?

---

### Record what was not verified

Examples:

```text
Verified: scripts parse; unit tests pass; sample workflow completes.
Not verified: full provider download; Docker build; cross-platform behavior.
```

Stating a verification boundary is more transparent than implying complete confidence.

---

## 7. Record material decisions

Code shows what the computer executes. It may not explain why a source, threshold, model, join, or exclusion was chosen.

Record decisions that affect research interpretation, for example:

- why one provider was chosen;
- why a country or year was excluded;
- why a particular unit conversion is valid;
- why a model is descriptive rather than causal;
- why a warning was accepted;
- why generated code was revised or rejected; and
- what AI assistance contributed to a material method.

Suitable locations include:

- README or implementation documentation;
- data dictionary or provenance record;
- analysis report;
- issue or pull request;
- architecture or decision record;
- commit body; and
- methods or AI-use statement.

---

### Avoid unsupported AI-generated citations

For every paper, dataset, quotation, version, licence, and URL suggested by an AI system:

1. locate the authoritative source;
2. confirm title, authors, date, identifier, and content;
3. read enough of the source to ensure it supports the claim;
4. cite the source rather than the AI response; and
5. state uncertainty when verification is incomplete.

Do not include a citation merely because it looks plausible.

---

## 8. Commit one reviewed outcome

Stage selected paths rather than every visible change:

```bash
git add scripts/validate-data.R reports/data-validation.qmd
```

Review the staged snapshot:

```bash
git diff --staged
git status --short
```

Then commit:

```bash
git commit -m "Validate the teaching-data candidate key"
```

---

### Commit content

A strong commit contains one coherent outcome and, when applicable:

- implementation;
- tests or validation;
- documentation;
- configuration needed to run it; and
- intentional updates to generated release artifacts.

Avoid mixing an AI-assisted feature with unrelated formatting, dependency updates, personal files, or previous uncommitted work.

---

### Commit message

The title should describe the completed outcome:

```text
Validate the teaching-data candidate key
```

Use the body to explain motivation, important decisions, and verification:

```text
Validate the teaching-data candidate key

- stop when area, item, element, year, and unit are duplicated
- preserve duplicate evidence rather than deleting rows automatically
- document the check in the validation report

Verified with the fixed teaching snapshot and a modified duplicate fixture.
```

The message does not need to say "AI generated." Disclosure belongs in the appropriate research or project record. The commit message should explain the project change.

---

### A commit is not a certificate of correctness

The commit records the accepted state. Preserve test evidence and review processes appropriate to the project's risks.

---

## 9. Integrate and publish deliberately

Push the working branch when it is ready to share:

```bash
git push -u origin feature/validate-teaching-data
```

Use the project's review mechanism, such as a pull or merge request. A reviewer should be able to see:

- the problem and intended outcome;
- the complete diff;
- verification evidence;
- scientific or data assumptions;
- known limitations; and
- any material AI assistance required by project policy.

---

### Before merging

Confirm:

- automated checks pass;
- human review is complete;
- conflicts are resolved intentionally;
- documentation and environment files agree with the implementation;
- no sensitive material is present;
- generated results were recreated from the reviewed code; and
- the target branch is correct.

---

### Tag research milestones

After review and integration, a tag can identify a milestone:

```bash
git tag -a analysis-v1 -m "Reviewed analysis used for the first report"
git push origin analysis-v1
```

A useful research release also identifies data, environment, configuration,
and outputs; the tag alone cannot recreate ignored external data.

---

## Handling data, secrets, and external services

### Classify information before sharing it

Do not send data to an AI service merely because the interface accepts an upload. Determine:

- whether the data contain personal, confidential, proprietary, embargoed, or location-sensitive information;
- which contractual, ethical, institutional, and legal conditions apply;
- whether the service retains or uses submitted content;
- where processing occurs;
- whether the approved account and settings are being used; and
- whether a de-identified, synthetic, aggregate, or local alternative is sufficient.

If the conditions are unclear, do not provide the data.

---

### Never include secrets in prompts or tracked files

Secrets include:

- passwords;
- API tokens;
- private SSH keys;
- cloud credentials;
- database connection strings;
- private endpoints; and
- confidential participant identifiers.

Use approved secret-management mechanisms. If a secret appears in a prompt, log, diff, or commit, stop, revoke or rotate it, and follow the project's incident process. Deleting the current line does not remove copies from Git history or an external service.

---

### Review agent tool use

An agent may be able to:

- read files outside the intended task;
- access the network;
- install or execute software;
- modify Git state;
- call external APIs; or
- send messages and create remote resources.

Grant only the capabilities needed for the bounded task. Review external and destructive actions before authorizing them.

---

## Research integrity and disclosure

### Verify facts and sources

AI systems are not bibliographic databases or authoritative documentation. Verify claims with primary sources and record the sources actually consulted.

---

### Preserve intellectual contribution accurately

AI systems do not take responsibility for research. Human authors and contributors remain accountable for the work. Describe human contributions using the study's authorship policy; a contributor-role taxonomy can help record conceptualization, data curation, formal analysis, software, validation, visualization, and writing.

---

### Follow applicable disclosure policy

Institutions, journals, funders, courses, and collaborators may require different forms of AI-use disclosure. Determine the policy before submission.

A useful disclosure can identify:

- tool or service;
- relevant date or version when available;
- purpose of use;
- material affected;
- human review and verification; and
- known reproducibility limitations.

Do not make a generic statement that implies every generated claim was checked if it was not.

---

### Keep methods and writing distinct

Assistance with grammar is different from assistance that changes:

- the research question;
- source selection;
- data exclusions;
- transformation logic;
- statistical methods;
- interpretation; or
- reported conclusions.

The more methodologically important the contribution, the stronger the need for explicit records, verification, and disclosure.

---

## What cannot be reproduced exactly

### Model and service changes

A provider may update a model or interface without making the old version available. Tool behavior, safety rules, search results, and context handling may change.

---

### Nondeterminism

The same apparent request can produce different results. A saved prompt does not guarantee replay.

---

### Hidden context

System instructions, retrieved context, account settings, tool outputs, or conversation truncation may not be visible or exportable.

---

### External state

An agent may use websites, package registries, APIs, or databases that later change.

---

### Practical response

Preserve what the project controls:

- accepted code and documentation;
- Git commit and branch;
- input identities and source records;
- environment lockfiles or images;
- configuration and parameters;
- verification code and results;
- concise decision records; and
- tool/model metadata that is actually available and relevant.

State what cannot be reconstructed. Reproducibility is strengthened by honest boundaries, not by claiming exact replay when it is impossible.

---

## Failure and recovery

### The agent changed unrelated files

Inspect first:

```bash
git status --short
git diff
```

Do not discard changes until you know whether they belong to you or another collaborator. Ask the agent to separate or revert only its own changes when the ownership is clear. Avoid destructive repository-wide resets.

---

### The proposed approach is wrong

Reject it. An AI response is a proposal, not sunk cost. Preserve a short decision note only if the failed approach is scientifically or operationally important.

---

### Tests pass but results changed unexpectedly

Stop and compare:

- input checksums and versions;
- row counts and keys;
- environment changes;
- model parameters and random seeds;
- warnings and logs;
- numerical summaries; and
- the exact diff.

Passing tests may not cover the relevant scientific behavior.

---

### A citation cannot be verified

Remove it or replace it with a verified source. Do not invent missing bibliographic details.

---

### A sensitive value was exposed

Stop using the value, rotate or revoke the credential if applicable, notify the responsible person, and follow institutional incident procedures. Do not rely on deleting the local file alone.

---

### The session context is lost

Use the repository as the recovery point:

```bash
git status
git log --oneline --decorate -10
git show --stat HEAD
```

Read the README, task record, relevant diff, and latest verification evidence. A concise session summary can help, but the committed repository remains the authoritative accepted state.

---

## Application to the maize-yield project

### Scenario

Ask an AI coding agent to add one data-management check without changing the raw teaching snapshot.

Suitable task:

> Review `metadata/faostat-data-dictionary.csv` against the columns in
> `data/input/faostat-maize-yield-sample.csv`. Add a validation failure when the
> input contains an undocumented column or the dictionary describes a column
> absent from the input. Do not edit either CSV. Update the validation report
> and explain the limitation of column-name agreement.

---

### Starting checks

```bash
cd maize-yield-project
git status
git branch --show-current
git log --oneline -5
```

Create a branch:

```bash
git switch -c feature/validate-dictionary-coverage
```

---

### Invariants

- The fixed teaching CSV must remain byte-identical.
- The dictionary must not be rewritten automatically.
- A mismatch must stop or clearly fail validation.
- Matching names do not prove correct definitions or units.
- No package dependency should be added unless justified and approved.

---

### Review

```bash
git status --short
git diff
sha256sum data/input/faostat-maize-yield-sample.csv
```

Confirm that the source checksum still matches the recorded provenance.

---

### Verification

At minimum:

1. parse the changed R and Quarto files;
2. run validation with the fixed snapshot and dictionary;
3. test a temporary dictionary missing one input variable;
4. test a temporary dictionary containing one extra variable;
5. confirm the raw CSV did not change; and
6. inspect the report explanation.

Do not modify tracked source evidence to create a failure fixture. Use a temporary copy outside the tracked raw-data path or a test fixture designed for that purpose.

---

### Commit

```bash
git add scripts/validate-data.R reports/data-validation.qmd
git diff --staged
git commit -m "Validate dictionary coverage of teaching data"
```

The commit should record the accepted validation behavior. If AI assistance was methodologically material, add the appropriate project or course disclosure.

---

## Practice exercise

Choose one small improvement in your maize-yield repository:

- clarify one README instruction;
- add one validation assertion;
- add one test for a helper function;
- improve one error message; or
- document one analysis assumption.

Complete these steps:

1. Confirm a clean or understood starting state.
2. Create a descriptive branch.
3. Write a bounded task brief with invariants and verification.
4. Ask an AI assistant to inspect or implement the task.
5. Review every changed and untracked file.
6. Run suitable software, data, and scientific checks.
7. Record one material decision and one limitation.
8. Stage only the intended files.
9. Review the staged diff.
10. Commit with an outcome-focused message.
11. Write a short AI-use record if the assistance materially affected the research or if course policy requires it.

---

### Reflection

Answer:

- Which part of the AI output did you reject or revise?
- Which verification provided the strongest evidence?
- What remained unverified?
- Could another learner recover the accepted state without the chat transcript?
- Did the interaction expose any information that should not have been shared?

---

## Completion checklist

### Before AI assistance

- [ ] The repository, branch, status, remote, and starting commit are known.
- [ ] Existing changes are understood and preserved.
- [ ] The task has one bounded outcome.
- [ ] In-scope and out-of-scope paths are stated.
- [ ] Scientific and data invariants are explicit.
- [ ] Required verification is defined before implementation.
- [ ] Sensitive data and credential constraints are understood.
- [ ] External and destructive actions require explicit authorization.

---

### During AI assistance

- [ ] The agent receives only the context and permissions needed.
- [ ] Diagnosis is separated from modification when the cause is uncertain.
- [ ] Assumptions and unverified claims are made visible.
- [ ] Authoritative sources are used for facts, APIs, methods, and citations.
- [ ] No secret or restricted information is placed in prompts or files.

---

### Before committing

- [ ] `git status` and the complete unstaged diff were reviewed.
- [ ] Every new file was opened and inspected.
- [ ] Unrelated changes are excluded.
- [ ] Software checks pass at an appropriate level.
- [ ] Data grain, keys, units, missingness, and coverage remain valid.
- [ ] Scientific assumptions and interpretations were reviewed by a human.
- [ ] Facts, references, quotations, and links were verified.
- [ ] Verification boundaries and remaining limitations are recorded.
- [ ] The staged diff contains exactly the intended outcome.
- [ ] The commit message explains the completed change.

---

### Before publishing or merging

- [ ] Review and automated checks are complete.
- [ ] Environment, data provenance, and configuration are reproducible.
- [ ] No credential, personal data, or restricted material is present.
- [ ] AI-use disclosure follows course, institutional, funder, and journal policies.
- [ ] Accepted results can be traced to a repository commit and input state.
- [ ] Claims do not exceed the verified evidence.

---

## Check your understanding

1. Why does saving a prompt not guarantee exact reproduction of an AI response?
2. What should be the primary reproducible object in an AI-assisted coding workflow?
3. Why should a commit correspond to a reviewed outcome rather than every AI interaction?
4. Name three kinds of evidence that Git does not provide by itself.
5. What should you inspect in addition to `git diff` before staging?
6. Give an example of a software test passing while the research method remains invalid.
7. Which information about an AI interaction should be preserved when it materially changes a research method?
8. Why is a Git branch not a security boundary?
9. What should you do when an AI system proposes a paper or quotation?
10. How would you recover after losing the conversation context?

---

## Further resources

### Version control

- [Pro Git: Recording Changes to the Repository](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository.html)
  explains the working tree, staging area, and commit cycle.
- [Pro Git: Branches in a Nutshell](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell.html)
  explains how branches isolate lines of development.
- [The Turing Way: Version Control](https://book.the-turing-way.org/reproducible-research/vcs/)
  places version control in a reproducible-research workflow.

---

### Responsible AI and research practice

- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
  provides a voluntary framework for governing, mapping, measuring, and
  managing AI risks.
- [NIST Generative AI Profile](https://doi.org/10.6028/NIST.AI.600-1)
  addresses risks that are specific to or intensified by generative AI.
- [CRediT contributor roles](https://credit.niso.org/contributor-roles-defined/)
  provide a vocabulary for describing human contributions such as software,
  validation, data curation, formal analysis, and writing. CRediT does not
  determine authorship.

---

### Reproducibility and documentation

- [The Turing Way: Overview of Reproducible Research](https://book.the-turing-way.org/reproducible-research/overview.html)
  connects version control, computational environments, data, and research
  documentation.
- [Documentation tools for data management](04_03_data_management_tools.md) explains how Markdown,
  YAML, CSV, and SHA-256 checksums support structured, reviewable project
  records.
