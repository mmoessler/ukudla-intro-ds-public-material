# 1.6) A reproducible AI-assisted research workflow

---

- Last Update: 2026-09-03
- Source: [01_06_version_control_and_collaboration_ai_assisted_research_workflow.md](/learning-modules/intro-ds-module/01_06_version_control_and_collaboration_ai_assisted_research_workflow.md)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [What reproducibility means here](#what-reproducibility-means-here)
  - [Reproducibility is not automatic replay](#reproducibility-is-not-automatic-replay)
  - [Git records accepted states, not truth](#git-records-accepted-states-not-truth)
- [AI assists; researchers remain responsible](#ai-assists-researchers-remain-responsible)
  - [Human review is an activity, not a button](#human-review-is-an-activity-not-a-button)
- [The review–verify–commit loop](#the-reviewverifycommit-loop)
- [Prompting for reviewable work](#prompting-for-reviewable-work)
  - [Give a concrete outcome](#give-a-concrete-outcome)
  - [Separate diagnosis from modification](#separate-diagnosis-from-modification)
  - [State invariants](#state-invariants)
  - [Ask for evidence](#ask-for-evidence)
- [Further detail](#further-detail)

---

## Learning objectives

After completing this guide, you should be able to:

- explain how Git supports transparent AI-assisted research without making AI output automatically trustworthy;
- break work into bounded, reviewable tasks;
- establish permission, data, and file boundaries before using an agent;
- inspect and verify AI-generated changes before accepting them; and
- distinguish reproducible repository state from exact reproduction of an AI conversation.

For the full step-by-step procedure, checklists, and a worked example, see the
[AI-assisted research workflow reference](01_07_version_control_and_collaboration_ai_assisted_research_workflow_reference.md).

---

## Place in the session

This page extends the Version Control session:

```text
Why Git?  →  Setup  →  Repository setup  →  Collaborative workflow
                                                     ↓
                                  Reproducible AI-assisted workflow
```

Review these pages first:

- [Why use Git and GitHub?](01_01_version_control_and_collaboration_motivation.md)
- [Create your maize yield repository](01_04_version_control_and_collaboration_repository_setup.md)
- [A collaborative Git workflow](01_05_version_control_and_collaboration_application.md)

This guide assumes that you can inspect a repository, create a branch, review changes, stage selected files, commit, and push.

The goal is not to recommend one AI service. The workflow applies to coding agents, chat interfaces, editor assistants, and command-line tools. Their capabilities and interfaces differ, but the researcher still has to define, review, verify, and document the work.

---

## What reproducibility means here

An AI-assisted research change is reproducible when another person can recover the accepted project state and understand how the result was produced and checked.

Useful evidence includes:

- the repository commit;
- input-data identity and provenance;
- scripts, configuration, and environment definitions;
- the accepted code and documentation changes;
- commands or procedures used for verification;
- important assumptions and human decisions; and
- relevant information about AI assistance when it materially affected the work.

---

### Reproducibility is not automatic replay

An AI response can depend on:

- model and model version;
- system instructions and available tools;
- conversation context;
- repository state;
- external search or service results;
- random or nondeterministic generation;
- provider-side updates; and
- time-dependent information.

Even with the same prompt, another run may produce a different response. Therefore, the primary reproducible object is normally the **reviewed repository state and its evidence**, not the expectation that an agent will regenerate the same text byte for byte.

---

### Git records accepted states, not truth

Git can show what changed and preserve a reviewed snapshot. It does not prove that:

- code is correct;
- an analysis is scientifically valid;
- data are representative;
- a citation exists;
- a model result is unbiased;
- a licence permits reuse; or
- the researcher understood the change.

Version control supports auditability. Testing, validation, domain knowledge, and human judgment establish whether a change should be accepted.

---

## AI assists; researchers remain responsible

Treat an AI system as a fallible tool that can propose, explain, transform, and inspect material. Do not treat it as an accountable researcher or an authoritative source.

AI output can contain:

- syntactically correct but logically wrong code;
- invented functions, packages, papers, quotations, or URLs;
- hidden changes outside the requested scope;
- inappropriate statistical assumptions;
- insecure commands or credential handling;
- outdated information;
- code copied or closely derived from material with unclear licensing; and
- confident explanations unsupported by evidence.

The human researcher remains responsible for:

- defining the question;
- deciding which data and methods are appropriate;
- controlling access to files, services, and credentials;
- checking facts and references against authoritative sources;
- reviewing every accepted change;
- running and interpreting verification;
- deciding what to commit and publish; and
- reporting methods and limitations honestly.

---

### Human review is an activity, not a button

Selecting "accept," "apply," or "approve" in an interface is not sufficient review. Review means reading the diff, understanding the logic, checking the scientific meaning, running suitable verification, and resolving uncertainty before accepting the change.

---

## The review–verify–commit loop

Use this loop for each bounded change:

```text
Define one outcome
        ↓
Inspect repository and data boundaries
        ↓
Create or choose a branch
        ↓
Give the agent context, scope, and constraints
        ↓
Review the complete diff
        ↓
Verify software behavior and research meaning
        ↓
Revise or reject if necessary
        ↓
Stage selected files and review again
        ↓
Commit the reviewed outcome
```

An AI interaction does not automatically deserve a commit. A commit should represent one coherent, accepted project outcome. The loop expands into nine concrete steps:

1. **Begin from a known repository state.** Check `git status`, the current branch, remotes, and the starting commit before asking an agent to act.
2. **Define one bounded task**, describing an observable outcome, in-scope and out-of-scope files, and required verification — not a vague instruction such as "improve the project."
3. **Choose a safe working context**, normally a descriptively named branch. A branch isolates history; it is not a security boundary for an agent's filesystem or network access.
4. **Give the agent sufficient context and boundaries**: the objective, relevant paths, conventions, invariants, and which actions require confirmation.
5. **Inspect every change yourself** — `git status --short` and `git diff` — including untracked files and history-sensitive files such as lockfiles, `.gitignore`, and credentials.
6. **Verify behavior and research meaning** at the software, data, and scientific level; passing tests do not guarantee a valid method.
7. **Record material decisions** that affect research interpretation, such as why a source or exclusion was chosen, and verify any AI-suggested citation against its authoritative source.
8. **Commit one reviewed outcome**, staging only the intended paths and writing a message that explains the completed change, not that it was AI-assisted.
9. **Integrate and publish deliberately** through the project's normal review mechanism, confirming checks, documentation, and sensitive-material screening before merging.

See the [reference page](01_07_version_control_and_collaboration_ai_assisted_research_workflow_reference.md) for the full detail, command examples, and checklists behind each step, plus guidance on handling data and secrets, research-integrity disclosure, failure recovery, and a worked maize-yield example.

---

## Prompting for reviewable work

### Give a concrete outcome

Weak:

> Improve the analysis.

Better:

> Explain why the model evaluation may leak future information. Do not modify
> files. Cite the exact code paths involved and propose two fixes.

Better implementation request:

> Implement the approved time-based split in `scripts/predict-maize-yield.R`.
> Preserve the existing output schema, add a test for the split boundary, and
> update the report. Do not change package versions.

---

### Separate diagnosis from modification

When the cause is uncertain:

```text
Inspect and explain the failure first. Do not edit files yet.
```

After reviewing the diagnosis:

```text
Implement the selected fix and run the targeted checks.
```

This prevents an early guess from becoming an unreviewed code change.

---

### State invariants

Examples:

- raw data must remain unchanged;
- public function signatures must remain compatible;
- the output key must remain unique;
- the train/test boundary must remain 2017/2018;
- no new dependency may be added; and
- no network access is permitted.

---

### Ask for evidence

Useful request:

```text
After implementation, report changed files, verification commands, results,
remaining limitations, and any assumptions you could not verify.
```

Then independently inspect that evidence.

---

## Further detail

The [AI-assisted research workflow reference](01_07_version_control_and_collaboration_ai_assisted_research_workflow_reference.md)
covers each of the nine steps in full, including command examples, permission
boundaries, data and secrets handling, research-integrity and disclosure
requirements, what cannot be reproduced exactly, failure and recovery
patterns, a worked maize-yield example, a practice exercise, completion
checklists, and further resources.
