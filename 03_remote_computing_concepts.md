# Remote-computing concepts

---

- Last Update: 2026-08-28
- Source: [03_remote_computing_concepts.md](/learning-modules/intro-ds-module/03_remote_computing_concepts.md)

---

## Learning objectives

After reading this page, you should be able to:

- distinguish Linux, a terminal, a shell, and SSH;
- explain local and remote computing contexts;
- distinguish authentication from authorization;
- reason about paths, processes, pipes, and permissions; and
- identify safe practices for remote work.

---

## Computing contexts

The same project may be used on several computers:

```text
learner laptop → local container → remote server → shared infrastructure
```

Each context can have a different hostname, user account, filesystem, software
environment, and resource policy. Before running a consequential command, ask:

```text
Which computer? Which user? Which directory?
```

The commands `hostname`, `whoami`, and `pwd` answer these questions.

---

## Related terms

- **Linux** is an operating-system kernel and commonly refers to operating
  systems built around it.
- A **terminal** is an interface for entering and viewing text commands.
- A **shell** interprets commands; Bash is a common shell.
- The **command line** is the text-based interaction with a shell.
- **SSH** is a protocol and tool family for encrypted communication with
  remote services.

Opening a terminal does not necessarily create a remote session. Running a
shell in a local container also differs from running a shell on a remote
server.

---

## SSH connections

An SSH connection normally identifies:

- a client computer;
- a remote host and network address;
- a remote user account;
- an authentication method; and
- the service and host key being trusted.

Authentication establishes who is connecting. Authorization determines what
that account may do after connecting. A successful SSH login does not imply
permission to read every file or use every shared resource.

Public-key authentication uses a private key retained by the user and a public
key registered with the service. Private keys must not be shared or committed.
The server host key protects against connecting silently to the wrong host and
should be verified when first encountered or unexpectedly changed.

---

## Filesystems and paths

An absolute path begins at `/`; a relative path begins at the shell's current
working directory. The same textual path can identify different files on a
laptop, in a container, and on a server.

Useful inspection commands include:

```bash
pwd
ls -la
find . -maxdepth 2 -type f | sort
```

File permissions specify what the owner, group, and other users may read,
write, or execute. On shared infrastructure, project directories and sensitive
data should follow the institution's access policy rather than permissive
defaults.

---

## Processes, streams, and pipes

A process is a running program. Shell commands communicate through standard
input, standard output, and standard error. A pipe sends one command's standard
output to the next command:

```bash
find scripts -type f | sort
```

Redirection and pipes are powerful because they can overwrite files or pass
unexpected data onward. Inspect a command and its current directory before
running it, especially with elevated permissions or recursive operations.

Long-running remote work should use the infrastructure's recommended job
scheduler or session-management tool. An ordinary SSH connection ending may
also end processes attached to that session.

---

## Data transfer and execution are separate

SSH can support remote shells and secure file-transfer tools, but transferring
code does not reproduce its environment. A remote analysis still needs:

- versioned project files;
- recorded software dependencies;
- available and permitted data;
- appropriate compute resources; and
- documented execution commands.

Use Git for versioned source collaboration and approved transfer tools for data
that do not belong in Git.

---

## Continue with the applications

Use [the SSH application](03_remote_computing_ssh_application.md) to establish
and inspect a remote connection. Use
[the Linux application](03_remote_computing_linux_application.md) to navigate,
inspect, and combine command-line tools safely.

---

## Key message

Remote computing becomes manageable when the computer, identity, directory,
permissions, and software environment are made explicit before commands run.
