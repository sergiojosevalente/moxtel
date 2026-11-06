# Moxtel security

How Moxtel protects your data, and its limits.

## Principles

- **No cloud, no telemetry, no account.** Moxtel connects only to the servers you open (SSH, RDP, VNC, FTP, S3, …)
  and, when you check for updates, to github.com. Your session list is never synchronised anywhere.
- **Everything stays on this computer**, in `%APPDATA%\moxtel` (or the folder given with `-data`), readable only by you.
- **Secrets are never stored as plain text.**

## What is encrypted, and how

| Data | Protection |
|---|---|
| Sessions, folders, tunnels, macros, transfer profiles, settings (`config.dat`) | Windows DPAPI for your Windows account, with application-specific entropy. |
| Passwords, key passphrases, S3 secrets (`vault.json`) | AES-256-GCM. The data key is wrapped with a key derived from your **master password** with **Argon2id** (64 MiB, 3 passes, 4 threads, random 16-byte salt). Each entry is authenticated with its name. |
| "Remember on this PC" for the vault | The data key is protected with DPAPI; can be turned off at any time. |
| Command history for suggestions (`history.dat`) | DPAPI. Lines starting with a space and commands that look like they contain passwords or tokens are never stored. |
| Auto Transfer run history | DPAPI. |
| `known_hosts` | Host names are hashed (OpenSSH `HashKnownHosts` format). |
| Backups (`.moxbackup`) | Separate backup password → Argon2id → AES-256-GCM. |

Showing or exporting a saved password always asks for the master password again, even while the vault is open.

## Connections

- SSH host keys are checked against `known_hosts`; new keys must be confirmed and changed keys show a man-in-the-middle warning.
- Background runs (schedules, folder watch, command line, Task Scheduler) use only saved passwords and already-trusted host keys, and never show a dialog.
- FTPS verifies TLS certificates (TLS 1.2 or newer). FTP, Telnet, Rsh, plain VNC and XDMCP are **not encrypted** — use them only on trusted networks or through an SSH tunnel.
- Updates: Moxtel installs an update only if `SHA256SUMS` carries a valid Ed25519 signature from the release key built into the app, and the installer matches its SHA-256.

## Limits

- Malware running as your Windows user can open DPAPI-protected data, and can read passwords while the vault is unlocked. Lock the vault and turn off "remember on this PC" for the strongest protection (scheduled transfers then need you to unlock it).
- Terminal logs and recordings, if you turn them on, contain whatever appears on screen and are stored as normal files.

## Reporting a vulnerability

Please **do not open a public issue**. Use **Security → Report a vulnerability** on this repository
(GitHub private vulnerability reporting), or contact the author through https://sergiojosevalente.github.io/.
Include the Moxtel version and steps to reproduce. You will get an answer as soon as possible.
