# Apply `renv` to the example project

---

- Last Update: 2026-08-28
- Source: [02_reproducible_environments_renv_application.md](/learning-modules/intro-ds-module/02_reproducible_environments_renv_application.md)

---

## Learning objectives

After completing this application, you should be able to:

- restore the R package environment from `renv.lock`;
- inspect whether the library and lockfile agree;
- run project code using the restored library; and
- explain when a lockfile should be updated.

---

## Before you begin

Work from the root of the standalone example repository. Confirm the location
and inspect its state:

```bash
pwd
git status
R --version
```

Do not begin by changing the lockfile. First reproduce the environment that the
project already records.

---

## Inspect the environment record

The repository contains:

```text
renv.lock
renv/
.Rprofile
```

`renv.lock` is the machine-readable package record. The `renv/` directory
contains project infrastructure, while `.Rprofile` activates `renv` when R
starts in the project.

Inspect the lockfile without editing it:

```bash
git log --oneline -- renv.lock
git status --short renv.lock
```

---

## Restore the package library

Start R from the repository root and run:

```r
renv::restore()
```

The first restore may download packages and therefore requires network access.
Review prompts before accepting them. When transfer must be minimized, preserve
the local `renv` cache and avoid deleting a working project library merely to
demonstrate restoration.

Check the result:

```r
renv::status()
sessionInfo()
```

`renv::status()` should explain whether the project library, lockfile, and
declared dependencies agree.

---

## Run the project in the restored environment

Exit R and run a lightweight project command, or execute the documented
pipeline when its data requirements are available:

```bash
Rscript scripts/run-all.R
```

The command should use the project library activated by `renv`. Record the
command, input state, and any external-data requirements when reporting the
result.

---

## Change dependencies deliberately

When the project genuinely requires a new or updated package:

1. create a focused Git branch or commit context;
2. install the intended package;
3. run the relevant checks or analysis;
4. inspect `renv::status()`;
5. call `renv::snapshot()` only when the new state is intentional; and
6. review the `renv.lock` diff before committing it.

```r
renv::snapshot()
```

Do not snapshot unrelated packages simply because they happen to be installed.

---

## Verification checklist

- [ ] R starts with the project `renv` environment active.
- [ ] `renv::status()` reports the expected state.
- [ ] The project command uses the restored package library.
- [ ] No unintended lockfile changes were created.
- [ ] Any intentional lockfile change was reviewed and tested.

---

## Handoff to Docker

The restored package layer does not describe every system dependency. Continue
with [the Docker application](02_reproducible_environments_docker_application.md)
to reproduce the broader execution context.
