# Reproducible-environment concepts

---

- Last Update: 2026-08-28
- Source: [02_reproducible_environments_concepts.md](/learning-modules/intro-ds-module/02_reproducible_environments_concepts.md)

---

## Learning objectives

After reading this page, you should be able to:

- define a computational environment;
- distinguish package, language, system, and infrastructure layers;
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

---

## `renv` manages the R package layer

`renv` creates a project-specific R package library and records package
versions and sources in `renv.lock`.

Two operations form the central workflow:

- `renv::snapshot()` records the packages used by the project; and
- `renv::restore()` installs the state described by the lockfile.

The lockfile belongs in version control. The project library normally does not.
Lockfile changes should be reviewed because they alter the environment that
collaborators and automated systems will restore.

`renv` does not record the complete operating system, every system library,
hardware, credentials, or remote services.

---

## Docker records a broader execution context

A Dockerfile is a recipe for building a container image. The image can record:

- a base operating-system image;
- system packages and command-line tools;
- R and other language runtimes;
- project files and dependency installation; and
- the default command and working directory.

A container is a running instance of an image. Rebuilding an image and starting
a new container are different actions. Bind mounts can connect selected host
directories to the container, allowing data or results to persist after the
container stops.

Docker improves portability, but tags such as `latest`, changing external
repositories, hardware differences, and network services can still introduce
variation.

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
with the analysis code.

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

Use [the `renv` application](02_reproducible_environments_renv_application.md)
to restore and verify the project-local R environment. Then use
[the Docker application](02_reproducible_environments_docker_application.md)
to build and run the broader execution environment.

---

## Key message

Record dependencies at the layer where they arise: `renv` describes the R
package state, while Docker describes a broader system and execution context.
