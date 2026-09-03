# 3.3) Use SSH with GitHub and remote servers

---

- Last Update: 2026-08-02
- Source: [03_03_remote_computing_ssh_application.md](/learning-modules/intro-ds-module/03_03_remote_computing_ssh_application.md)

---

## Learning objectives

After completing this guide, you should be able to:

- inspect an SSH setup without exposing private keys, and test GitHub authentication;
- confirm which Git remote a repository uses, and how GitHub SSH differs from a server login;
- open and close an SSH session on an authorized remote server; and
- recognize common SSH security warnings and connection errors.

---

## Two connections with different outcomes

This guide uses SSH in two ways:

```text
Git operations ──SSH──► github.com       repository data, no general shell
ssh command    ──SSH──► remote server    interactive remote shell
```

Both use encrypted connections and may share the same local key pair, but they connect to different services and grant different capabilities.

---

## Part 1: Review SSH access to GitHub

The complete initial key setup is in the
[version-control setup page](01_03_version_control_and_collaboration_setup.md).
The following steps review and test that setup.

---

### 1. Inspect names, not private-key contents

```bash
ls -al ~/.ssh
```

Common files:

```text
id_ed25519       private key — never share or display its contents
id_ed25519.pub   public key — this is the key registered with GitHub
known_hosts      identities of hosts accepted previously
config           optional per-host client settings
```

A student may have differently named keys; do not create a replacement merely because the names differ. If an SSH agent is in use, `ssh-add -l` lists the keys it holds — `The agent has no identities` means no key is loaded there, not that no key exists on disk.

---

### 2. Test GitHub authentication

```bash
ssh -T git@github.com
```

On the first connection, SSH may show a host fingerprint; compare it with [GitHub's published SSH key fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints) before accepting it.

A successful test identifies your GitHub username and explains that GitHub does not provide shell access. The test can still return exit status `1` — for this special GitHub command, the authentication message is the result that matters. The literal username in the command is `git`, not your GitHub username; GitHub determines your account from the public key used during authentication.

---

### 3. Inspect and test this repository's Git remote

From the repository root:

```bash
git remote -v
git remote get-url origin
git ls-remote origin
```

An SSH URL has the form `git@github.com:OWNER/REPOSITORY.git` — for the public
maize yield repository, `git@github.com:mmoessler/maize-yield-project.git`. If
the repository setup exercise changed `origin` to your own GitHub repository,
the owner should instead be your GitHub username. Remote names are local
labels, not guarantees; read the URLs before pulling or pushing.

`git ls-remote` reads the remote references without creating a commit or push.
Authentication to GitHub can succeed even when your account lacks permission
to write to a particular repository.

---

## Part 2: Connect to a remote computing server

Only connect to systems for which you have authorization. Obtain the server
hostname, username, port (if not 22), VPN requirement, authentication
method, and official host-key fingerprint from the instructor or
administrator; the examples below use placeholders.

---

### 1. Open the connection

```bash
ssh USER@HOST
```

For example, a course might provide a command resembling `ssh student@login.example.edu`. If the administrator specifies another port:

```bash
ssh -p PORT USER@HOST
```

On the first connection, verify the displayed host fingerprint through a trusted channel before typing `yes` — the accepted key is recorded in `~/.ssh/known_hosts`. Enter a password only at the SSH password or key-passphrase prompt; the terminal normally displays no characters while you type it.

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

These identify the remote computer, account, directory, time, load, and storage. Do not assume the remote server has the same files, R version, or packages as your laptop or the Docker image. Follow the server's policies for storage locations, software modules, data, and jobs — on a shared cluster, substantial work commonly belongs in a batch scheduler rather than on the login node.

---

### 3. Run a command and end the session

SSH can run one command without an interactive session (`ssh USER@HOST hostname`, output in the local terminal). Leave an interactive session with `exit` or `Ctrl-D`, and confirm the prompt has returned to the local computer.

---

## Optional: alias and file transfer

An entry in `~/.ssh/config` can store non-secret connection details for reuse:

```text
Host course-server
    HostName HOST
    User USER
    Port PORT
    IdentityFile ~/.ssh/id_ed25519
```

Omit `Port` for port 22, use the real key path from setup, connect with
`ssh course-server`, and protect the files with `chmod 700 ~/.ssh` and
`chmod 600 ~/.ssh/config`. Never put a password or private-key contents in
the configuration file.

`scp` copies files over SSH — confirm source and destination carefully:

```bash
scp LOCAL_FILE USER@HOST:~/
scp USER@HOST:~/REMOTE_FILE .
```

The colon separates the remote host from its path; without it, `scp` may
treat both arguments as local. For large datasets, use the institution's
recommended transfer service instead of assuming `scp` is suitable.

---

## Keys, jobs, and disconnection

Keep personal credentials on the host computer; do not copy a private key into the maize yield image or commit one to Git — cloning and version control belong on the host.

A foreground process may stop when its SSH session disconnects, so do not assume a closed laptop leaves an analysis running. Use the mechanism approved for the server — a job scheduler such as Slurm, a persistent terminal tool such as `tmux` if allowed, or a managed service — and avoid long or resource-heavy work on a shared login node unless its policy explicitly permits it.

---

## Troubleshooting

| Problem | Response |
|---|---|
| `Permission denied (publickey)` | Run `ssh-add -l` and `ssh -v USER@HOST` for diagnostics, then confirm username and key registration with the administrator. For GitHub, see the [SSH troubleshooting guide](https://docs.github.com/en/authentication/troubleshooting-ssh/error-permission-denied-publickey). |
| `Could not resolve hostname` | Check for a typo and confirm whether a VPN is required. |
| Connection timed out or refused | Server, port, network, VPN, or firewall may be unavailable — repeated key creation will not fix it. |
| `REMOTE HOST IDENTIFICATION HAS CHANGED` | Stop. Contact the administrator and verify the new fingerprint before changing `known_hosts`. |
| GitHub authentication works but `git push` fails | Authentication and authorization are separate — check `git remote -v` and confirm write access. |

---

## Check your work

- I can explain why `ssh -T git@github.com` does not open a shell.
- I can identify the owner and repository in an SSH Git remote.
- I know which SSH key file must remain private, and verify a new host fingerprint through an official source.
- I use `hostname`, `whoami`, and `pwd` after a remote login, and know long work may need a scheduler or persistent session tool.

---

## Videos

- [SSH key authentication for GitHub](https://www.youtube.com/watch?v=ZgARMqR3qq8) — GitHub; generating, registering, and testing a key.
- [SSH Crash Course](https://www.youtube.com/watch?v=hQWRp-FdTpc) — Traversy Media; remote login, keys, configuration, and file transfer.

---

## Further reading

- [Connecting to GitHub with SSH — GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [OpenSSH server documentation — Ubuntu](https://ubuntu.com/server/docs/openssh-server)
- [`ssh` manual page — OpenBSD](https://man.openbsd.org/ssh)
