# 2.2) Reproducible-environment concepts

---

- Last Update: 2026-08-29
- Source: [02_02_reproducible_environments_concepts.md](/learning-modules/intro-ds-module/02_02_reproducible_environments_concepts.md)

---

## Learning objectives

After reading this page, you should be able to:

- define a computational environment and its main layers;
- explain the `renv` snapshot/restore workflow and what it records;
- distinguish a Docker image from a container and explain what a Dockerfile records;
- explain the complementary roles of `renv` and Docker;
- distinguish recording an environment from reproducing an analysis; and
- identify limitations that environment tools cannot remove.

---

## What is a computational environment?

A computational result depends on more than source code. Relevant conditions
can include:

```text
project code and configuration
R packages and package versions
R and publishing-tool versions
system libraries and operating system
hardware and external services
```

Differences at any layer can change whether code runs or what it produces. A
reproducible environment records enough of these conditions to recreate the
intended execution context.

Environment isolation keeps one project's dependencies from unexpectedly
changing another project. Portability makes the recorded context usable on a
different computer. Neither property guarantees that the scientific method is
valid.

Every language ecosystem provides its own tools for the package layer: R
projects typically use `renv`, Python projects often use `venv`, Conda, or
Poetry, and other languages have comparable managers. The mechanism differs,
but the goal is the same — keep a project's dependencies separate from other
projects and describe them in a machine-readable form. A language-level
environment manager generally does not fully specify the host operating
system, system libraries, drivers, or external command-line programs; that is
the layer Docker addresses.

---

## `renv` manages the R package layer

`renv` creates a project-specific R package library and records package
versions and sources in `renv.lock`. The core workflow moves between the
project library and the lockfile:

```text
project code ──► dependency discovery
                       │
                       ▼
               project library
                  │         ▲
         snapshot │         │ restore
                  ▼         │
                  renv.lock
```

Two operations form the central workflow:

- `renv::snapshot()` records the packages used by the project; and
- `renv::restore()` installs the state described by the lockfile.

`renv::status()` compares the code, library, and lockfile and reports where
they disagree. The lockfile belongs in version control. The project library
normally does not: it is large and machine-specific, so it is reconstructed
from the lockfile rather than shared directly. Lockfile changes should be
reviewed because they alter the environment that collaborators and automated
systems will restore.

`renv` does not record the complete operating system, every system library,
hardware, credentials, or remote services.

---

## Docker records a broader execution context

A Docker **image** is a packaged, layered description containing files,
binaries, libraries, and configuration; a **container** is an isolated
process started from an image:

```text
Dockerfile ──docker build──► image ──docker run──► container
```

A Dockerfile is a recipe for building that image. It can record:

- a base operating-system image;
- system packages and command-line tools;
- R and other language runtimes;
- project files and dependency installation, for example restoring an `renv`
  library; and
- the default command and working directory.

Rebuilding an image and starting a new container are different actions: the
image is a versioned description, and a container is one run of it. Bind
mounts can connect selected host directories to a container, allowing data or
results to persist after the container stops. Containers are lighter than
full virtual machines because they share the host kernel, but they still
provide a controlled user-space environment. Docker improves portability, but
tags such as `latest`, changing external repositories, hardware differences,
and network services can still introduce variation.

---

## `renv` and Docker are complementary

| Concern | `renv` | Docker |
|---|---|---|
| Project-specific R packages | Primary responsibility | Can install from the lockfile |
| R package sources and versions | Recorded in `renv.lock` | Inherits the restored package state |
| Operating-system packages | Not managed | Recorded in the Dockerfile |
| Runtime isolation | R package library | Process and filesystem context |
| Typical artifact | `renv.lock` | `Dockerfile` and image |

Using both tools avoids asking either one to describe a layer it was not
designed to manage. The lockfile and Dockerfile should be versioned together
with the analysis code: in the maize yield project, `renv.lock` records R
4.3.3 and the R package dependencies, the `Dockerfile` starts from
`rocker/verse:4.3.3` and restores those packages during the build, and the
resulting image runs `scripts/run-all.R`. Git stores both recipes; `renv` and
Docker reconstruct the environments they describe.

---

## Reproducible environment versus reproducible analysis

Recreating an environment is necessary for many analyses but is not sufficient.
The result also depends on:

- available and correctly interpreted data;
- deterministic or recorded random processes;
- complete workflow instructions;
- accessible external services;
- appropriate analytical assumptions; and
- responsible interpretation.

An environment can reproduce a coding error or a scientifically invalid
analysis exactly. Reproducibility makes work inspectable; it does not certify
correctness.

---

## Continue with the applications

Use [the `renv` application](02_04_reproducible_environments_renv_application.md)
to restore and verify the project-local R environment. Then use
[the Docker application](02_06_reproducible_environments_docker_application.md)
to build and run the broader execution environment.

---

## Key message

Record dependencies at the layer where they arise: `renv` describes the R
package state, while Docker describes a broader system and execution context.
