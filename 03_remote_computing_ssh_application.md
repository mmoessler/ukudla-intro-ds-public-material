# Use SSH with GitHub and remote servers

---

- Last Update: 2026-08-02
- Source: [03_remote_computing_ssh_application.md](/learning-modules/intro-ds-module/03_remote_computing_ssh_application.md)

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Two connections with different outcomes](#two-connections-with-different-outcomes)
- [Part 1: Review SSH access to GitHub](#part-1-review-ssh-access-to-github)
  - [1. Inspect names, not private-key contents](#1-inspect-names-not-private-key-contents)
  - [2. Test GitHub authentication](#2-test-github-authentication)
  - [3. Inspect this repository's Git remote](#3-inspect-this-repositorys-git-remote)
  - [4. Test repository access without changing it](#4-test-repository-access-without-changing-it)
- [Part 2: Connect to a remote computing server](#part-2-connect-to-a-remote-computing-server)
  - [1. Open the connection](#1-open-the-connection)
  - [2. Confirm where you are](#2-confirm-where-you-are)
  - [3. Run a single safe remote command](#3-run-a-single-safe-remote-command)
  - [4. End the session](#4-end-the-session)
- [Optional: Make a reusable SSH alias](#optional-make-a-reusable-ssh-alias)
- [Optional: Transfer a small file](#optional-transfer-a-small-file)
- [Where should SSH keys live?](#where-should-ssh-keys-live)
- [Remote jobs and disconnections](#remote-jobs-and-disconnections)
- [Troubleshooting](#troubleshooting)
  - [Problem 01](#problem-01)
  - [Problem 02](#problem-02)
  - [Problem 03](#problem-03)
  - [Problem 04](#problem-04)
  - [Problem 05](#problem-05)
- [Check your work](#check-your-work)
- [Videos](#videos)
- [Further reading](#further-reading)

---

## Learning objectives

After completing this guide, you should be able to:

- inspect an SSH setup without exposing private keys;
- test SSH authentication to GitHub;
- confirm which Git remote a repository uses;
- explain how a GitHub SSH connection differs from a server login;
- open and close an SSH session on an authorized remote server; and
- recognize common SSH security warnings and connection errors.

---

## Two connections with different outcomes

This guide uses SSH in two ways:

```text
Git operations ──SSH──► github.com       repository data, no general shell
ssh command    ──SSH──► remote server    interactive remote shell
```

Both use encrypted connections and may use the same local key pair. They connect to different services and grant different capabilities.

---

## Part 1: Review SSH access to GitHub

The complete initial key setup is in the
[version-control setup page](01_version_control_and_collaboration_setup.md).
The following steps review and test
that setup.

---

### 1. Inspect names, not private-key contents

```bash
ls -al ~/.ssh
```

Common files include:

```text
id_ed25519       private key — never share or display its contents
id_ed25519.pub   public key — this is the key registered with GitHub
known_hosts      identities of hosts accepted previously
config           optional per-host client settings
```

A student may have differently named keys. Do not create a replacement merely because the names differ.

If an SSH agent is in use, list the keys it currently holds:

```bash
ssh-add -l
```

`The agent has no identities` means no key is loaded in that agent; it does not prove that no key exists on disk.

---

### 2. Test GitHub authentication

```bash
ssh -T git@github.com
```

On the first connection, SSH may show a host fingerprint. Compare it with [GitHub's published SSH key fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints) before accepting it.

A successful test identifies your GitHub username and explains that GitHub does not provide shell access. The test can still return exit status `1`; for this special GitHub command, the authentication message is the important result.

The literal username is `git`, not your GitHub username:

```text
ssh -T git@github.com
       ^^^
```

GitHub determines your account from the public key used during authentication.

---

### 3. Inspect this repository's Git remote

From the repository root:

```bash
git remote -v
git remote get-url origin
```

An SSH URL has this form:

```text
git@github.com:OWNER/REPOSITORY.git
```

For the public maize yield repository, it is:

```text
git@github.com:mmoessler/maize-yield-project.git
```

If the repository setup exercise changed `origin` to your own GitHub repository, the owner should instead be your GitHub username. Some workflows keep the course repository as a second remote named `upstream`. Remote names are local labels; read the URLs before pulling or pushing.

---

### 4. Test repository access without changing it

```bash
git ls-remote origin
```

This reads the remote references and does not create a commit or push a change. Authentication to GitHub can succeed even when your account lacks permission to write to a particular repository.

---

## Part 2: Connect to a remote computing server

Only connect to systems for which you have authorization. Obtain these details from the instructor or administrator:

- server hostname or IP address;
- your server username;
- whether a VPN or institutional network is required;
- the SSH port, if it is not the default port 22;
- the required authentication method; and
- the official host-key fingerprint.

The examples below use placeholders. Do not enter them literally.

---

### 1. Open the connection

```bash
ssh USER@HOST
```

For example, a course might provide a command resembling:

```text
ssh student@login.example.edu
```

If the administrator specifies another port:

```bash
ssh -p PORT USER@HOST
```

On the first connection, verify the displayed host fingerprint through a trusted channel before typing `yes`. The accepted key is recorded in `~/.ssh/known_hosts`.

Enter a password only at the SSH password or key-passphrase prompt. The terminal normally displays no characters while you type it.

---

### 2. Confirm where you are

After login:

```bash
hostname
whoami
pwd
date
uptime
df -h
```

These commands identify the remote computer, account, directory, time, load, and storage. Do not assume that the remote server has the same files, R version, or packages as your laptop or the Docker image.

Follow the server's policies for storage locations, software modules, data, and computing jobs. On a shared cluster, substantial work commonly belongs in a batch scheduler rather than on the login node.

---

### 3. Run a single safe remote command

SSH can run one command without opening an interactive session:

```bash
ssh USER@HOST hostname
```

The command runs remotely; its output appears in the local terminal.

---

### 4. End the session

```bash
exit
```

You can also press `Ctrl-D`. Confirm that the prompt has returned to the local computer before continuing.

---

## Optional: Make a reusable SSH alias

After a successful direct connection, an entry in `~/.ssh/config` can store non-secret connection details:

```text
Host course-server
    HostName HOST
    User USER
    Port PORT
    IdentityFile ~/.ssh/id_ed25519
```

Omit `Port` when the server uses port 22. Use the real key path supplied or chosen during setup. Then connect with:

```bash
ssh course-server
```

Protect the configuration and SSH directory:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
```

Do not put a password or private-key contents in the configuration file.

---

## Optional: Transfer a small file

`scp` copies over SSH. Confirm the source and destination carefully.

Copy a local file to your remote home directory:

```bash
scp LOCAL_FILE USER@HOST:~/
```

Copy a remote file into the current local directory:

```bash
scp USER@HOST:~/REMOTE_FILE .
```

The colon separates the remote host from its remote path. Without it, `scp` may interpret both arguments as local paths. For large research datasets, use the institution's recommended transfer service instead of assuming `scp` is suitable.

---

## Where should SSH keys live?

Keep personal credentials on the host computer. Do not copy a private key into the maize yield image or commit one to Git.

Running Git from inside a disposable container usually does not require GitHub credentials because code should be cloned and version-controlled on the host. SSH-agent forwarding and mounting an SSH directory have security implications and are outside this introductory exercise.

---

## Remote jobs and disconnections

A foreground process may stop when its SSH session disconnects. Do not assume that closing a laptop leaves an analysis running.

Use the mechanism approved for the server:

- a job scheduler such as Slurm on a computing cluster;
- a persistent terminal tool such as `tmux`, if allowed; or
- a managed service supplied by the institution.

Ask the administrator which option is expected. Do not run long or resource-heavy analysis on a shared login node unless its policy explicitly permits it.

---

## Troubleshooting

---

### Problem 01

Problem: `Permission denied (publickey)`

The server did not accept a presented key. Check:

```bash
ssh-add -l
ssh -v USER@HOST
```

`-v` produces diagnostic details and may reveal which keys were attempted. It does not fix permissions. Confirm the username, public-key registration, and required identity with the administrator.

For GitHub, follow the [GitHub SSH troubleshooting guide](https://docs.github.com/en/authentication/troubleshooting-ssh/error-permission-denied-publickey).

---

### Problem 02

Problem: `Could not resolve hostname`

Check the hostname for a typing error and confirm whether the institutional VPN or network is required.

---

### Problem 03

Problem: Connection timed out or was refused

The server, port, network, VPN, or firewall may be unavailable. Verify the supplied connection details; repeated key creation will not fix a network failure.

---

### Problem 04

Problem: `REMOTE HOST IDENTIFICATION HAS CHANGED`

Stop. A server may have been rebuilt, but the warning can also indicate an attack or the wrong host. Contact the administrator and verify the new fingerprint before changing `known_hosts`. Do not automatically delete the warning.

### Problem 05

Problem: GitHub authentication works but `git push` fails

SSH authentication and repository authorization are separate. Check:

```bash
git remote -v
git branch --show-current
git status
```

Confirm that the remote is your intended repository and that your GitHub account has write access.

## Check your work

- I can explain why `ssh -T git@github.com` does not open a shell.
- I can identify the owner and repository in an SSH Git remote.
- I know which SSH key file must remain private.
- I verify a new host fingerprint through an official source.
- I use `hostname`, `whoami`, and `pwd` after a remote login.
- I can leave a remote session with `exit`.
- I know that long work may require a scheduler or persistent session tool.

---

## Videos

- [SSH key authentication for GitHub](https://www.youtube.com/watch?v=ZgARMqR3qq8) — GitHub; a focused review of generating, registering, and testing a key.
- [SSH Crash Course](https://www.youtube.com/watch?v=hQWRp-FdTpc) — Traversy Media; covers remote login, keys, configuration, and file transfer.
- [Introduction to Linux – Full Course for Beginners](https://www.youtube.com/watch?v=sWbUDq4S6Y8) — freeCodeCamp; a longer course that places remote command-line work in a broader Linux context.

---

## Further reading

- [Connecting to GitHub with SSH — GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [Testing your SSH connection — GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection)
- [OpenSSH server documentation — Ubuntu](https://ubuntu.com/server/docs/openssh-server)
- [`ssh` manual page — OpenBSD](https://man.openbsd.org/ssh)
- [`scp` manual page — OpenBSD](https://man.openbsd.org/scp)
