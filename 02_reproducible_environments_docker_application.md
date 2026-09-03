# Apply Docker to the example project

---

- Last Update: 2026-08-28
- Source: [02_reproducible_environments_docker_application.md](/learning-modules/intro-ds-module/02_reproducible_environments_docker_application.md)

---

## Learning objectives

After completing this application, you should be able to:

- explain what the project Dockerfile records;
- build the example-project image;
- distinguish image contents from bind-mounted host files;
- run the pipeline in a container; and
- decide when an image must be rebuilt.

---

## Before you begin

Complete the [Docker setup page](02_reproducible_environments_docker_setup.md)
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
secrets.

---

## Build the image

```bash
docker build -t maize-yield-project .
```

Docker reuses cached build layers when their instructions and inputs have not
changed. Rebuild the image after changing the Dockerfile, `renv.lock`, or source
files copied during the build. A change only in a bind-mounted host directory
does not necessarily require rebuilding.

Inspect the image:

```bash
docker image ls maize-yield-project
```

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

The image supplies the code and software environment. The mounts allow selected
host files to be read or written. Changes made elsewhere inside a container are
discarded when a `--rm` container exits.

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
