# Version control and collaboration concepts

---

- Last Update: 2026-08-28
- Source: [01_version_control_and_collaboration_concepts.md](/learning-modules/intro-ds-module/01_version_control_and_collaboration_concepts.md)

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

`.gitignore` documents recurring exclusions, but it does not remove a file that
has already been committed. Sensitive information requires prevention rather
than reliance on later cleanup.

---

## Relationship to the other pages

[Why use version control and collaboration?](01_version_control_and_collaboration_motivation.md)
introduces the problem. The
[application page](01_version_control_and_collaboration_application.md) applies
these concepts in a collaborative Git workflow. The setup pages cover local
tools and repository initialization.

---

## Key message

Version control makes project changes inspectable, recoverable, and shareable.
Its value depends on deliberate file selection, coherent commits, and a clear
understanding of local and remote history.
