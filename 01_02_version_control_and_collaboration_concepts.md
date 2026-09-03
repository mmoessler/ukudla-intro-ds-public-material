# 1.2) Version control and collaboration concepts

---

- Last Update: 2026-08-28
- Source: [01_02_version_control_and_collaboration_concepts.md](/learning-modules/intro-ds-module/01_02_version_control_and_collaboration_concepts.md)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Git records project history](#git-records-project-history)
- [GitHub supports shared work](#github-supports-shared-work)
- [Four states of a tracked change](#four-states-of-a-tracked-change)
- [A minimal example](#a-minimal-example)
- [Branches and synchronization](#branches-and-synchronization)
- [Track deliberately](#track-deliberately)
- [Relationship to the other pages](#relationship-to-the-other-pages)
- [Key message](#key-message)

---

## Learning objectives

After reading this page, you should be able to:

- distinguish Git from GitHub;
- explain working-directory, staging-area, commit, branch, and remote states;
- distinguish saving, committing, pushing, and pulling;
- explain why small, coherent commits support review and recovery; and
- identify files that should not be placed in version control.

---

## Git records project history

Git is a distributed version-control system. A Git repository contains the
current project files and a history of recorded project states. Each
collaborator normally has a complete local repository, including its history.

Git does not record every save automatically. The researcher decides which
changes belong together and records them as a **commit**. A useful commit:

- represents one coherent change;
- contains only intended files;
- leaves the project in an understandable state; and
- has a message that explains the purpose of the change.

The commit history is therefore a sequence of meaningful project states, not a
backup of every keystroke.

---

## GitHub supports shared work

GitHub hosts Git repositories and adds collaboration services such as access
control, issues, pull requests, reviews, and automation. Git and GitHub are
related but not interchangeable:

| Git | GitHub |
|---|---|
| Runs locally | Provides an online service |
| Records commits and branches | Hosts shared repositories |
| Works without a network for most operations | Requires network access for synchronization |
| Manages version history | Adds review and coordination tools |

A local repository can exist without GitHub. GitHub cannot replace the local
Git concepts needed to understand the history.

---

## Four states of a tracked change

A typical change moves through four states:

```text
working directory → staging area → local history → remote history
       edit            git add       git commit      git push
```

- The **working directory** contains the files currently being edited.
- The **staging area** selects the exact content intended for the next commit.
- The **local history** contains commits already recorded on the computer.
- The **remote history** contains commits shared through a service such as
  GitHub.

`git status`, `git diff`, and `git diff --staged` make these states visible.
Inspecting them is part of the method, not merely troubleshooting.

---

## A minimal example

Consider a single change to `README.md`:

```bash
git status
# nothing to commit, working tree clean

echo "Document the maize-yield teaching dataset." >> README.md
git status
# Changes not staged for commit: README.md

git add README.md
git status
# Changes to be committed: README.md

git commit -m "Document the maize-yield teaching dataset"
git push origin main
```

Each command narrows the change from an edited file to a permanent, shared
entry in the project history. Skipping `git add` before `git commit` records
nothing; skipping `git push` after `git commit` keeps the change on the local
computer only, invisible to collaborators who only look at GitHub.

---

## Branches and synchronization

A branch is a movable name pointing to a line of commits. Branches allow work
to develop without immediately changing another line of work. When histories
diverge, Git must integrate them through a merge or rebase.

The central synchronization operations are:

- `git fetch`: obtain remote history without integrating it;
- `git pull`: obtain and integrate remote history; and
- `git push`: publish local commits to a remote repository.

A push can be rejected when the remote branch contains commits that the local
branch does not yet contain. This protects shared history from being silently
overwritten.

Two common ways to integrate diverging histories are a **merge**, which
creates a new commit joining both histories, and a **rebase**, which replays
local commits on top of the updated remote history. A merge preserves the
exact order of events and is the default and safer choice for beginners; a
rebase produces a straighter, easier-to-read history but rewrites commit
identifiers. Avoid rebasing commits that have already been pushed and shared
with collaborators — rewriting shared history forces everyone else to
reconcile their own copy with the rewritten one.

---

## Track deliberately

Version control is appropriate for source code, documentation, configuration,
small metadata files, and environment lockfiles. It is usually inappropriate
for:

- passwords, tokens, private keys, or other secrets;
- confidential or restricted data;
- large downloaded datasets that can be acquired reproducibly;
- generated caches and package libraries; and
- outputs that can be rebuilt reliably and need not be distributed in Git.

Tracking generated or reproducible artifacts inflates repository size,
produces noisy diffs on every rebuild, and creates a false impression that
the committed file — rather than the script that produces it — is the
authoritative source.

`.gitignore` documents recurring exclusions, but it does not remove a file that
has already been committed. Sensitive information requires prevention rather
than reliance on later cleanup.

---

## Relationship to the other pages

[Why use version control and collaboration?](01_01_version_control_and_collaboration_motivation.md)
introduces the problem. The
[application page](01_05_version_control_and_collaboration_application.md) applies
these concepts in a collaborative Git workflow. The setup pages cover local
tools and repository initialization.

---

## Key message

Version control makes project changes inspectable, recoverable, and shareable.
Its value depends on deliberate file selection, coherent commits, and a clear
understanding of local and remote history.
