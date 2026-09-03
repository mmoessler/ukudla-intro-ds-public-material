# 2.6) Apply Docker to the example project

---

- Source: [02_06_reproducible_environments_docker_application.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/02_06_reproducible_environments_docker_application.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/02_06_reproducible_environments_docker_application.md)
- Feedback: [Topic 02: Reproducible Environments](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/3)

---

## Learning objectives

After completing this application, you should be able to:

- explain what the project Dockerfile records;
- build the example-project image and interpret common build failures;
- distinguish image contents from bind-mounted host files;
- run the pipeline in a container; and
- decide when an image must be rebuilt.

---

## Before you begin

Complete the [Docker setup page](02_05_reproducible_environments_docker_setup.md)
and work from the example repository root.

```bash
docker --version
git status
```

Building may download a base image and packages. Reuse an existing image when
the Dockerfile, lockfile, and files copied into the image have not changed.

---

## Inspect the build recipe

Read the Dockerfile before building it:

```bash
sed -n '1,240p' Dockerfile
```

Identify:

- the base image;
- installed system dependencies;
- the working directory;
- files copied into the image;
- how the R package environment is restored; and
- the default command.

The Docker build context determines which project files can become part of the
image. `.dockerignore` should exclude unnecessary data, results, caches, and
secrets — anything listed there is never sent to the Docker build process at
all, which keeps the image smaller and prevents accidentally baking local
data or credentials into a shareable image.

---

## Build the image

```bash
docker build -t maize-yield-project .
```

Docker reuses cached build layers when their instructions and inputs have not
changed, and reruns a layer — along with every layer after it — as soon as
one of its inputs changes. This is why a well-ordered Dockerfile installs
rarely-changing system dependencies before it copies frequently-edited
analysis code: editing a script should not force system packages to
reinstall. Rebuild the image after changing the Dockerfile, `renv.lock`, or
source files copied during the build. A change only in a bind-mounted host
directory does not necessarily require rebuilding.

Inspect the image:

```bash
docker image ls maize-yield-project
```

---

## When a build fails

A failed build is usually one of a few kinds of problem:

- **A layer instruction fails**, for example a system package name that does
  not exist for the base image's distribution. The build output identifies
  the failing instruction; fix the Dockerfile rather than working around the
  failure inside a running container.
- **`renv::restore()` fails inside the build.** The same causes as a local
  restore failure apply (missing system library, network problem, R version
  mismatch); the fix belongs in the Dockerfile or the lockfile, not in a
  container patched after the fact.
- **The build context is too large or includes unwanted files.** Check
  `.dockerignore` first.

---

## Run the pipeline

Persist data and outputs by mounting explicit host directories:

```bash
docker run --rm -it \
  -v "$(pwd)/data:/work/data" \
  -v "$(pwd)/results:/work/results" \
  -v "$(pwd)/figures:/work/figures" \
  -v "$(pwd)/reports:/work/reports" \
  maize-yield-project bash
```

Inside the container, confirm the context and execute the documented workflow:

```bash
pwd
Rscript scripts/run-all.R
```

The image supplies the code and software environment; anything `COPY`-ed into
it during the build is fixed at build time. The `-v` mounts instead connect
selected host directories to paths inside the running container, so data can
be read and results can be written back to the host. Changes made elsewhere
inside a container — anywhere not covered by a mount — are discarded when a
`--rm` container exits.

---

## Choosing what to mount

Mount only the host directories the workflow actually needs to read or
write; avoid bind-mounting the whole repository over the image's working
directory, which would silently replace the code baked into the image with
whatever happens to be on the host and defeat the point of building a
versioned image in the first place. The example mounts above cover `data`,
`results`, `figures`, and `reports` because those are the directories the
pipeline reads from or writes to — the code itself is expected to come from
the image, not from a mount.

---

## Stopping and reusing containers

`--rm` removes the container as soon as it exits, which is appropriate for a
one-off pipeline run. Omitting `--rm` keeps the container around after it
stops, which can be useful while debugging: `docker ps -a` lists stopped
containers, `docker start -ai <container>` resumes one, and `docker rm
<container>` removes it once it is no longer needed. A stopped container
still occupies disk space until it is removed.

---

## Validate the run

After leaving the container, inspect the mounted output directories on the
host:

```bash
git status --short
find results figures reports -maxdepth 2 -type f | sort
```

Confirm that:

- expected outputs exist;
- output ownership and permissions are usable on the host;
- no secrets or unintended data entered the image; and
- the command and image-building inputs are documented.

---

## Verification checklist

- [ ] The Dockerfile and `.dockerignore` were inspected.
- [ ] The image builds successfully.
- [ ] The container starts in the expected working directory.
- [ ] The project pipeline runs using the image environment.
- [ ] Required outputs persist through explicit mounts.
- [ ] You can explain whether the next source change requires rebuilding.

---

## Key message

An image is a versioned execution environment; a container is one run of that
image. Rebuild when image inputs change, and use deliberate mounts for data and
outputs that must persist.
