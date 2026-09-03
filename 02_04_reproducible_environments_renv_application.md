# 2.4) Apply `renv` to the example project

---

- Source: [02_04_reproducible_environments_renv_application.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/02_04_reproducible_environments_renv_application.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/02_04_reproducible_environments_renv_application.md)
- Feedback: [Topic 02: Reproducible Environments](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/3)

---

## Learning objectives

After completing this application, you should be able to:

- restore the R package environment from `renv.lock`;
- inspect whether the library and lockfile agree;
- run project code using the restored library;
- interpret common `renv::restore()` and `renv::status()` messages; and
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
project already records. Restoring before editing anything establishes a known
baseline: if something fails later, you can tell whether the cause is your
change or an environment that was never correctly restored in the first
place.

---

## Inspect the environment record

The repository contains:

```text
renv.lock
renv/
.Rprofile
```

`renv.lock` is the machine-readable package record: for every recorded
package it lists a name, version, and source (for example CRAN or GitHub).
The `renv/` directory contains project infrastructure such as the local
package cache and the bootstrap script, while `.Rprofile` activates `renv`
automatically whenever R starts in the project directory — this is why
opening the project in RStudio or starting `R` from its root already uses the
project library rather than your personal one.

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

`renv::restore()` reads `renv.lock`, compares it with the packages already
present in the project library, and installs, removes, or leaves packages
unchanged so that the library matches the lockfile exactly. The first restore
may download packages and therefore requires network access. Review prompts
before accepting them. When transfer must be minimized, preserve the local
`renv` cache and avoid deleting a working project library merely to
demonstrate restoration — a warm cache lets `renv` reuse already-downloaded
package builds instead of fetching them again.

Check the result:

```r
renv::status()
sessionInfo()
```

`renv::status()` explains whether the project library, lockfile, and declared
dependencies agree. A message such as "No issues found" means the three are
consistent. If it instead reports packages that are recorded in the lockfile
but missing from the library, run `renv::restore()` again; if it reports
packages that are used in the code but not recorded in the lockfile, do not
snapshot them yet — first decide, as below, whether adding that dependency is
intentional.

A typical clean report looks like:

```text
No issues found -- the project is in a consistent state.
```

A report that needs attention instead names specific packages, for example:

```text
The following package(s) are recorded in the lockfile but are not
installed:

  dplyr [1.1.4]

Use `renv::restore()` to install the packages recorded in the lockfile.
```

Read the message before acting on it. "Recorded in the lockfile but not
installed" calls for `renv::restore()`; "used in the project but not
recorded in the lockfile" calls for a decision about whether to add that
dependency, not an automatic snapshot.

---

## Where restored packages live

`renv` does not download a fresh copy of every package for every project.
Installed packages are stored once in a shared, version-specific cache
outside the project (by default under a per-user cache directory), and the
project library links to that cache. Restoring a package that another
`renv` project on the same machine already restored at the same version is
therefore usually fast, because `renv` reuses the cached build instead of
reinstalling it. Deleting a project's `renv/library` directory does not
delete the shared cache, and clearing the shared cache does not remove
anything from `renv.lock`: the lockfile is the durable record, while the
cache and the project library are both disposable and can be rebuilt from
it.

---

## Troubleshooting a failed restore

A restore can fail for reasons unrelated to the project code itself:

- **Missing system libraries.** Some R packages depend on system-level
  libraries when compiling from source. The error message usually names the
  missing library; install it with the operating system's package manager
  rather than editing the lockfile.
- **Network or mirror problems.** A transient failure reaching a package
  repository can often be resolved by retrying `renv::restore()`.
- **R version mismatch.** `renv.lock` records the R version the project was
  built with. A materially different local R version can make some packages
  fail to install; report the mismatch rather than silently switching to a
  different lockfile.

Investigate the actual error before changing the lockfile — a failed restore
is a signal to fix the environment, not a reason to regenerate the recorded
package versions.

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

`renv::snapshot()` updates `renv.lock` to match the current project library,
so it should only run after a change you intend to keep. When the project
genuinely requires a new or updated package:

1. create a focused Git branch or commit context;
2. install the intended package;
3. run the relevant checks or analysis;
4. inspect `renv::status()`;
5. call `renv::snapshot()` only when the new state is intentional; and
6. review the `renv.lock` diff before committing it.

```r
renv::snapshot()
```

Do not snapshot unrelated packages simply because they happen to be
installed — `renv::status()` will list them, and a snapshot records
everything currently in the library, not only the package you meant to add.

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
with [the Docker application](02_06_reproducible_environments_docker_application.md)
to reproduce the broader execution context.
