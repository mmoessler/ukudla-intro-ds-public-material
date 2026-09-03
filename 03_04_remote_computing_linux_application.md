# 3.4) Use Linux inside the maize yield container

---

- Last Update: 2026-08-02
- Source: [03_04_remote_computing_linux_application.md](/learning-modules/intro-ds-module/03_04_remote_computing_linux_application.md)

---

## Learning objectives

After completing this guide, you should be able to:

- start and enter the project's Docker container, and tell the host from the container;
- navigate, inspect, and combine project files with basic Linux commands and pipes;
- understand which container changes persist; and
- make and save a small practice edit with Vim.

---

## Before you begin

Open a terminal in the repository root, check the context, and build the
image if needed (or rebuild after changing the `Dockerfile` or `renv.lock`):

```bash
pwd
git status
docker version
docker build -t maize-yield-project .
```

The image uses Linux and R 4.3.3 from `rocker/verse`. Its working directory is `/work`, and its default command runs `scripts/run-all.R`.

---

## Method 1: Create and enter a container directly

Prepare the host directories that will receive data and results, then create a container and enter its Bash shell:

```bash
mkdir -p data-raw data-processed figures reports

docker run --rm -it \
  -v "$(pwd)/data-raw:/work/data-raw" \
  -v "$(pwd)/data-processed:/work/data-processed" \
  -v "$(pwd)/figures:/work/figures" \
  -v "$(pwd)/reports:/work/reports" \
  maize-yield-project \
  bash
```

`--rm` removes the container after it stops, `-it` allocates an interactive terminal, `-v host:container` bind-mounts a host directory, and `bash` replaces the default analysis command with an interactive shell.

---

## Establish your context

Run these commands inside the container:

```bash
pwd
whoami
hostname
cat /etc/os-release
R --version
```

`pwd` should report `/work`; `hostname` shows the container's generated identifier. Compare with the same commands on the host, and when in doubt ask: which prompt, computer, directory, user?

---

## Navigate through the project

List and move between directories, then find files without changing them:

```bash
ls -la
cd scripts && pwd && ls
cd .. && cd reports && cd -
find . -maxdepth 2 -type f
find scripts -type f -name "*.R"
```

`/work/scripts` is an **absolute path**; `scripts` is a **relative path**
starting from the current directory. `.` is the current directory, `..` the
parent, and `cd -` returns to the previous one. Linux paths are
case-sensitive: `README.md` and `readme.md` are different names.

---

## Inspect text and data, and combine commands

```bash
cat .Rprofile                          # display a short file
head -n 10 scripts/run-all.R           # first lines
tail -n 10 scripts/run-all.R           # last lines
less README.md                         # scrollable viewer — /word to search, q to quit
wc -l scripts/*.R                      # count lines
grep -R -n "renv" docs                 # search text
head -n 5 data-processed/maize-yield-panel.csv  # peek at processed data
```

A pipe, `|`, sends the output of the command on its left to the input of the
command on its right — useful for combining the tools above, for example
`find scripts -type f | sort` or `grep -R -n "maize" scripts | head`. This
does not create a file; redirection (`>` overwrites, `>>` appends) does.
Do not practise redirection on tracked project files — an accidental `>` can
replace their contents.

---

## Inspect storage and processes

```bash
df -h        # filesystem capacity
du -sh /work # project directory size
ps           # processes in the current shell
```

Many commands describe their options with `--help` (e.g. `grep --help`); manual pages may not be installed in a minimal container.

---

## Run project operations

Check the restored R environment, inspect the pipeline before running it, and
run the complete analysis only when ready for the FAOSTAT download with the
output directories mounted:

```bash
Rscript -e 'renv::status()'
less scripts/run-all.R
Rscript scripts/run-all.R
```

The first stage downloads a large FAOSTAT archive. See the
[`renv` application](02_04_reproducible_environments_renv_application.md) and
[Docker application](02_06_reproducible_environments_docker_application.md) for
the local and container workflows.

---

## A brief Vim exercise

Vim is a modal command-line text editor. Check availability with `vim --version` (try `vi --version` otherwise); do not install software in the container for this exercise. Use a disposable file, `vim /tmp/vim-practice.txt`, so the exercise cannot alter the repository. Vim starts in **Normal mode**: press `i` for **Insert mode**, type `Maize yield analysis`, press `Esc`, then type `:w` and Enter to save and `:q` and Enter to quit.

Essential commands:

| Keys | Action |
|---|---|
| `i` | enter Insert mode |
| `Esc` | return to Normal mode |
| `:w` | save |
| `:q` | quit when no unsaved changes remain |
| `:wq` | save and quit |
| `:q!` | quit and discard unsaved changes |
| `u` | undo |
| `dd` | delete the current line |
| `/word` | search for `word` |
| `n` | move to the next search result |

`cat /tmp/vim-practice.txt` reads it back; it disappears with this `--rm` container.

---

## Persistence: image, container, and mounts

The project source was copied into the image at build time; edits to that copy exist only in the container and disappear with `--rm`. The bind-mounted directories (`data-raw`, `data-processed`, `figures`, `reports`) differ — changes there affect the host immediately, since a container is isolation, not a backup. Edit source on the host, record it with Git, and rebuild the image.

---

## Method 2: Enter an already running container

`docker exec` opens another process in an existing container — start one named in the background, then enter it:

```bash
docker run --rm -dit \
  --name maize-shell \
  -v "$(pwd)/data-raw:/work/data-raw" \
  -v "$(pwd)/data-processed:/work/data-processed" \
  -v "$(pwd)/figures:/work/figures" \
  -v "$(pwd)/reports:/work/reports" \
  maize-yield-project \
  bash

docker ps
docker exec -it maize-shell bash
```

Type `exit` to leave the added shell — the container keeps running. Stop it from the host with `docker stop maize-shell`; because it was created with `--rm`, Docker removes it after it stops. (To leave the *direct* container from Method 1, run `exit` at its shell prompt — this stops and removes it, but not the image or the mounted host files.)

---

## Troubleshooting

| Problem | Response |
|---|---|
| `docker: command not found` | Install Docker per the [Docker setup page](02_05_reproducible_environments_docker_setup.md), then open a new terminal. |
| `Unable to find image 'maize-yield-project:latest'` | Run `docker build -t maize-yield-project .` from the repository root. |
| A bind mount is empty or unexpected | Run `pwd` on the host before `docker run` — `$(pwd)` must be the repository root. |
| `vim: command not found` | Use `vi` if available, or do the exercise elsewhere; this does not block the analysis. |
| Permission errors in a mounted directory | Stop before using `sudo`. Record the command, path, and `ls -l` output, and ask the instructor. |

---

## Check your work

- I can identify whether a prompt is on the host or in a container.
- I can navigate, inspect files, and combine commands with pipes.
- I know which project directories are bind-mounted, and can save a disposable Vim file, leave, and stop the container cleanly.

---

## Videos

- [Bash/Terminal Crash Course for Beginners](https://www.youtube.com/watch?v=oxuRxtrO2Ag) — Traversy Media; shell, files, navigation, search, and piping.
- [Vim in 100 Seconds](https://www.youtube.com/watch?v=-txKSRn0qeA) — Fireship; a concise introduction to Vim's modal editing.

---

## Further reading

- [The Linux command line for beginners — Ubuntu](https://ubuntu.com/desktop/docs/en/latest/tutorial/the-linux-command-line-for-beginners/)
- [The Unix Shell lesson — Software Carpentry](https://swcarpentry.github.io/shell-novice/)
- [`docker container exec` reference — Docker Docs](https://docs.docker.com/reference/cli/docker/container/exec/)
