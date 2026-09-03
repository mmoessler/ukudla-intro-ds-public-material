# 2.1) Why use reproducible environments?

---

- Source: [02_01_reproducible_environments_motivation.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/02_01_reproducible_environments_motivation.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/02_01_reproducible_environments_motivation.md)
- Feedback: [Topic 02: Reproducible Environments](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/3)

---

## Learning objectives

After reading this guide, you should be able to:

- explain why code alone is not enough to reproduce an analysis;
- describe dependencies and dependency versions;
- distinguish an R project environment from a container; and
- identify important aspects of reproducibility that environment tools do not solve.

---

## The problem: "It works on my machine"

Imagine that one student runs the maize yield analysis successfully, but another student receives an error. Both students have the same scripts. The difference might be:

- a package is not installed;
- different package versions behave differently;
- the computers use different versions of R;
- a package needs a system library that is missing;
- the operating systems handle paths or fonts differently; or
- an external dataset or website has changed.

The scripts describe **what to do**, but an analysis also depends on the environment in which those scripts run.

---

## What is a computational environment?

A computational environment includes the software and configuration required
to execute a project: the language and its version, installed packages and
their versions, operating-system libraries, external tools such as Quarto and
Pandoc, environment variables, and the data and inputs the code reads. If
important parts of this environment are unspecified, a collaborator may have
to guess how to recreate it.

An R script that begins with `library(dplyr)` states that the project needs
`dplyr`, but not which version. Packages change over time: functions can gain
arguments, defaults can change, bugs can be fixed, and older behavior can be
removed. Installing the newest available package versions months later may
therefore produce different output or cause code to fail. Recording versions
turns an implicit assumption into explicit project metadata.

---

## Isolation, portability, and reproducibility

A useful environment aims to provide three properties:

- **Isolation:** changing packages for one project does not unexpectedly break another project.
- **Portability:** another person or computer can recreate the environment from shared instructions and metadata.
- **Reproducibility:** the recreated environment uses the intended dependency versions rather than whichever versions happen to be installed.

These properties overlap, but they are not identical. An isolated environment that exists only on one laptop is not yet portable. A portable description that always installs unpinned latest versions is not fully reproducible.

---

## Two complementary tools

Two kinds of tools address these properties at different layers.
[`renv`](https://rstudio.github.io/renv/) gives an R project its own package
library and records package versions in `renv.lock`, so the R package layer
can be restored exactly. Docker packages a broader execution context — an
operating system, system libraries, external tools, and the analysis code —
into a portable, runnable image. Python and other language ecosystems have
their own equivalents to `renv`, such as `venv`, Conda, or Poetry; the goal in
every case is the same: keep project dependencies separate from other
projects, and provide a machine-readable way to reconstruct them. The
[concepts page](02_02_reproducible_environments_concepts.md) explains how
`renv` and Docker divide this work and how they fit together.

---

## Why this matters in food-systems data science

Food-systems analyses may inform research, policy, monitoring, or operational decisions. They often combine data from different organizations and must be revisited after a semester, reporting cycle, or data revision.

A documented environment helps:

- a classmate reproduce a result;
- an instructor diagnose a problem;
- a research team rerun an analysis after new data arrive;
- reviewers understand the software context;
- an automated service execute the same workflow; and
- future maintainers update dependencies deliberately.

Reproducibility also improves learning: instead of spending class time on unexplained package differences, students can work from a known starting point.

---

## What these tools do not guarantee

`renv` and Docker improve computational reproducibility, but neither guarantees scientific validity or identical results in every situation. You must still manage source-data versions and provenance, random-number seeds, external web services and changing download URLs, secrets and credentials, hardware-specific calculations, locale and platform differences, undocumented manual steps, and statistical assumptions and data quality.

An environment can reproduce an incorrect analysis perfectly. Reproducibility makes work inspectable; it does not replace good methods.

---

## A practical reproducibility record

For a small data science project, record and version source scripts and
reports, a README with execution instructions, an R version and `renv.lock`,
`.Rprofile` and the `renv/` project files, a Dockerfile and `.dockerignore`,
data provenance and acquisition instructions, and checks that reveal whether
execution succeeded. Do not commit package libraries, passwords, tokens, or
confidential data.

---

## Check your understanding

1. Why can the same script behave differently on two computers?
2. What does `renv.lock` record, and why is it committed to Git?
3. What additional layer does Docker control compared with `renv`?
4. Name two reproducibility concerns that neither `renv` nor Docker solves by itself.

---

## Videos

These videos provide complementary explanations of the motivation and tools:

- [`renv`: Project Environments for R](https://www.youtube.com/watch?v=4wRiPG9LM3o) — Kevin Ushey from Posit introduces the "it worked before" problem, project-local libraries, isolation, portability, lockfiles, `snapshot()`, and `restore()`.
- [You should be using `renv`](https://www.youtube.com/watch?v=GwVx_pf2uz4) — E. David Aja at rstudio::conf explains how `renv` supports diagnosis, collaboration, and moving R projects between environments.
- [Docker in 100 Seconds](https://www.youtube.com/watch?v=Gjnup-PuquQ) — Fireship gives a concise visual explanation of containers, virtual machines, Dockerfiles, images, and running containers.
- [Docker Tutorial for Beginners](https://www.youtube.com/watch?v=3c-iBn73dDE) — TechWorld with Nana provides a longer introduction to the motivation for containers and demonstrates images, containers, commands, Dockerfiles, volumes, and Compose.

The Docker videos use web-application examples rather than an R analysis. Focus on the environment concepts: a Dockerfile describes an image, an image packages the environment, and a container is a running instance of that image.

---

## Further reading

- [Introduction to `renv`](https://rstudio.github.io/renv/articles/renv)
- [Using `renv` with Docker](https://rstudio.github.io/renv/articles/docker.html)
- [Docker: What is an image?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/)
- [The Turing Way: Reproducible environments](https://book.the-turing-way.org/reproducible-research/renv/renv)
