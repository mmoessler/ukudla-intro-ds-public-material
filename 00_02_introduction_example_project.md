# 0.2) Example Data Science Project

---

- Source: [00_02_introduction_example_project.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/00_02_introduction_example_project.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/00_02_introduction_example_project.md)
- Feedback: [Topic 00: Introduction](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/1)

---

## Outline

- [Outline](#outline)
- [Overall philosophy](#overall-philosophy)
- [Important caution about the example analysis](#important-caution-about-the-example-analysis)
- [Learning objectives](#learning-objectives)
- [Self-paced study](#self-paced-study)
- [Topic structure](#topic-structure)
  - [Part 0 – Introduction](#part-0--introduction)
  - [Part 1 – Tools and reproducibility](#part-1--tools-and-reproducibility)
  - [Part 2 – Data foundations](#part-2--data-foundations)
  - [Part 3 – Data analysis](#part-3--data-analysis)
- [Structure within a topic](#structure-within-a-topic)
- [Running project repository](#running-project-repository)
- [Data sources](#data-sources)
- [Completing the module](#completing-the-module)

---

## Overall philosophy

The module teaches reusable data-science practices and connects them through
**one continuous, reproducible example project**. The concepts are not limited
to that example: learners should be able to transfer them to laboratory data,
field experiments, field observations, surveys, administrative records, and
other secondary data.

**Running case study**:

❗**Understanding changes in maize yield in Southern Africa**

The running project makes dependencies between topics visible:

```text
Research Question
        ↓
Study Design and Data Collection
        ↓
Data Management
        ↓
Data Preparation
        ↓
Visualization
        ↓
Descriptive Analysis
        ↓
Explanatory or Predictive Analysis
        ↓
Interpretation
        ↓
Communication and Reproducibility
```

The project is an application and integration point, not the definition of the
course. A laboratory or field project may collect primary data rather than
download provider data, but it still needs version control, documentation,
validation, preparation, analysis, and reproducible communication.

---

## Important caution about the example analysis

> **Caution:** The maize-yield project is a developing teaching example, not a
> validated scientific analysis or a basis for practical decisions. The team
> is still working to identify defensible applications of the course topics,
> particularly for descriptive, explanatory, and predictive analysis. The
> current implementation contains substantial conceptual and analytical
> limitations.

Use the project to study workflow structure, reproducible implementation, and
critical evaluation. Do not interpret its estimates as established evidence
about maize-yield processes, causal effects, or future yields. Review the
assumptions, study design, measurement scale, model specification, diagnostics,
and scope of every analytical claim. The application and its conclusions may
change as these limitations are reviewed and corrected.

---

## Learning objectives

After completing the module, participants should be able to

- organize a reproducible data science project;
- recognize how study design and data origin affect an analysis;
- manage, document, integrate, and prepare data;
- create informative visualizations;
- describe distributions, relationships, and stability;
- distinguish explanation from prediction;
- fit, evaluate, and interpret simple statistical models; and
- communicate results in a reproducible report.

---

## Self-paced study

The module is asynchronous, self-paced, and location-independent. Topic
numbers express a recommended dependency order, not calendar weeks or fixed
meeting dates. Learners may distribute the work according to their prior
experience and available time.

Plan approximately **15 hours per topic** for the slides, detailed pages,
guided project work, independent practice, troubleshooting, reflection, and
documentation. This is a workload estimate rather than a deadline. Setup-heavy
topics may require more time initially, while experienced learners may move
more quickly through familiar material.

Follow the topics in order on a first pass because later work builds on earlier
artifacts and concepts. Returning to an earlier topic to revise a decision is
expected: a data-science workflow is iterative rather than strictly linear.

---

## Topic structure

### Part 0 – Introduction

**00 Introduction** establishes the course purpose, the distinction between
descriptive, explanatory, and predictive questions, and the complete reasoning
chain from a research question to a reproducible claim. It also introduces the
example project and the available development environments.

### Part 1 – Tools and reproducibility

1. **Version Control and Collaboration using Git and GitHub** — record
   meaningful project states, inspect changes, collaborate, and recover work.
2. **Reproducible Environments using `renv` and Docker** — record package and
   system dependencies and run analyses under controlled conditions.
3. **Remote Computing using SSH and Linux** — work safely and reproducibly in
   local containers and remote command-line environments.

At the end of Part 1, learners should be able to obtain a project, restore its
environment, inspect its history, and run it in an appropriate computing
context.

### Part 2 – Data foundations

4. **Data Management** — define observation grain, document meaning and
   provenance, validate fitness for purpose, and manage data responsibly.
5. **Data Integration** — align identifiers, time, space, units, and meaning
   when combining measurements or datasets.
6. **Data Preparation** — transform managed evidence into an analysis-specific
   dataset while preserving decisions, audits, and lineage.

These practices apply whether data originate in a laboratory, a designed field
experiment, an observational study, or an external provider.

### Part 3 – Data analysis

7. **Data Visualization** — map observations to defensible visual encodings for
   exploration and communication.
8. **Descriptive Analysis** — quantify distributions, variation, association,
   and evidence about stability or change.
9. **Explanatory Analysis** — formulate causal questions, distinguish design
   from estimation, and state the assumptions supporting an interpretation.
10. **Predictive Analysis** — define a prediction task, prevent leakage, compare
    models with baselines, and evaluate performance on realistic unseen data.

The three analytical purposes complement one another but are not
interchangeable. A descriptive association is not a causal effect, and a
causal coefficient is not evidence of predictive performance.

---

## Structure within a topic

Each topic follows a common core pathway:

1. **Slides** provide a concise overview and the central narrative.
2. **Motivation** explains the problem, relevance, and connection to preceding
   work.
3. **Concepts** introduces the terminology, distinctions, assumptions, and
   limitations needed for application.
4. **Application** turns the concepts into a reproducible workflow and asks the
   learner to inspect and interpret the resulting evidence.

Some topics include additional **setup**, **tools**, or **reference** pages.
Setup pages should be completed before the corresponding application.
Reference pages support later consultation and need not always be read from
beginning to end during the first pass.

For each topic, learners should:

- state what question or risk the topic addresses;
- inspect the relevant starting project state;
- predict what an operation should change;
- run or adapt the documented workflow;
- inspect outputs and unexpected results;
- explain important decisions and limitations; and
- record a small, reviewable project change.

---

## Running project repository

```text
maize-yield-project/
├── README.md
├── data/
│   ├── source/
│   ├── input/
│   └── derived/
├── metadata/
├── scripts/
├── reports/
├── figures/
├── results/
├── renv.lock
├── Dockerfile
└── maize-yield-project.Rproj
```

The example repository applies the course topics to changes in maize yield in
Southern Africa. It shows one coherent implementation, while the general
guidance explains which decisions must be adapted for other research designs
and data sources.

---

## Data sources

**Maize-yield outcomes**

- FAOSTAT crop and livestock products data

**Growing-season precipitation**

- CHIRPS v2 data accessed through ClimateSERV

The integrated teaching data include:

- maize production, harvested area, and yield;
- annual growing-season precipitation estimates;
- country and year identifiers; and
- source flags, documentation, dictionaries, and provenance records.

Both sources are secondary data. Other learners may apply the same module
structure to primary measurements collected under laboratory, experimental,
or observational protocols.

---

## Completing the module

Completion should be demonstrated through a reproducible project and a concise
report that connect:

- the research purpose and analytical objective;
- study design, data origin, population, and observation grain;
- data-management, integration, and preparation decisions;
- visual and numerical descriptions;
- an explanatory or predictive analysis appropriate to the question;
- interpretation, uncertainty, and limitations; and
- the project state, computational environment, and generated artifacts.

The sequence does not end with a universally "final" dataset or model. It ends
with a claim whose data, assumptions, code, and limitations can be inspected.
