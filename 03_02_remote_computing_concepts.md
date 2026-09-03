# 3.2) Remote-computing concepts

---

- Last Update: 2026-08-28
- Source: [03_02_remote_computing_concepts.md](/learning-modules/intro-ds-module/03_02_remote_computing_concepts.md)

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

| Context | Reached over a network | Shares the laptop's filesystem | Shares the laptop's software |
|---|---|---|---|
| Local shell | No | Yes | Yes |
| Local container | No | Only bind-mounted paths | No — its own image |
| Remote server (SSH) | Yes | No | No — the server's own environment |

A command that works in one context can fail, silently use different data, or
run against a different software version in another. Confirming the context
before acting is part of the method, not an optional courtesy.

---

## Related terms

- **Linux** is an operating-system kernel and commonly refers to operating
  systems built around it.
- A **terminal** is an interface for entering and viewing text commands.
- A **shell** interprets commands; Bash is a common shell.
- The **command line** is the text-based interaction with a shell.
- **SSH** is a protocol and tool family for encrypted communication with
  remote services.
- A **session** is one running shell together with the connection or terminal
  it is attached to. A remote session ends when that connection closes, even
  if the underlying command is still conceptually "your work."

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

## Sessions and disconnection

A foreground process started in an SSH session is normally a child of that
session's shell. When the network connection closes — deliberately or
because of a dropped network, a closed laptop lid, or a timeout — the shell
and its foreground children are commonly terminated as well. This is not a
bug in SSH; it follows from how the process is attached to the session that
launched it.

Two consequences follow. First, a long analysis should run under a mechanism
that deliberately detaches it from the login session, such as a job
scheduler or a persistent terminal multiplexer, rather than in the
foreground of an ordinary connection. Second, "the job is still running" is a
claim that should be verified on the remote system itself — for example by
listing processes or checking the scheduler's queue — rather than inferred
from the fact that the laptop is still open.

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

Use [the SSH application](03_03_remote_computing_ssh_application.md) to establish
and inspect a remote connection. Use
[the Linux application](03_04_remote_computing_linux_application.md) to navigate,
inspect, and combine command-line tools safely.

---

## Key message

Remote computing becomes manageable when the computer, identity, directory,
permissions, and software environment are made explicit before commands run.
