# Linux Privilege Escalation Checklist (First Pass)

**Phase:** 05-privesc-linux
**Platform:** Linux
**Source module(s):** 02 Getting Started, reinforced by the Nibbles box

## When to use

Immediately after landing a shell as a low-privileged user, before reaching for an automated enumeration script, so the manual checks are still understood rather than only read off a tool's output.

## Prerequisites

A shell on the target, any privilege level.

## Command

Checklist, run roughly in this order:

```bash
uname -a                                  # kernel/OS version, check against known CVEs
sudo -l                                   # NOPASSWD entries
find / -perm -4000 -type f 2>/dev/null    # SUID binaries
crontab -l; cat /etc/crontab              # cron jobs, then check file/dir write perms
cat ~/.bash_history                       # exposed creds/commands
find / -name "*.conf" -o -name "*.log" 2>/dev/null   # config/log scraping (scope this down in practice)
find / -name "id_rsa" 2>/dev/null         # readable SSH keys
```

| Part | What it does |
|---|---|
| `uname -a` | Kernel version, the basis for a kernel-exploit check |
| `sudo -l` | Lists what the current user can run as another user via sudo, and whether a password is required |
| `find / -perm -4000` | SUID binaries, see [weak-points](../../weak-points/README.md) for the GTFOBins habit this feeds into |
| `crontab -l` / `/etc/crontab` | Scheduled jobs; the interesting part is whether their script files or directories are writable by the current user |
| `~/.bash_history` | Often contains plaintext commands with embedded credentials |
| readable `id_rsa` | A private key readable by the current user that belongs to a higher-privileged account |

## Expected output

A short list of candidates to investigate individually: an outdated kernel version, a `sudo -l` entry, a SUID binary, a writable cron script, a credential in history or a config file, or a readable key.

## What failure looks like

Nothing turning up from any single check is a normal result, not a dead end. It means the path to escalation is elsewhere (installed software, a service misconfiguration, or something the automated scripts below would catch that a quick manual pass missed).

## Why it works

Each item in this checklist targets a distinct category of misconfiguration that grants a privilege boundary crossing:
- **Kernel exploits:** a bug in the kernel itself, independent of any application running on top of it.
- **`sudo -l` / SUID:** both are explicit, intentional privilege-escalation mechanisms (run-as-root by design) that become a vulnerability only when combined with a binary that can be abused to do something unintended, which is exactly what GTFOBins catalogs.
- **Cron jobs:** root-owned scheduled tasks that run with root's privileges; if the script file or its directory is writable by a lower-privileged user, that user can inject commands that root will later execute unattended.
- **Credentials in history/config/logs:** operator error, not a technical vulnerability, but extremely common and cheap to check.
- **SSH keys:** a private key readable due to a permissions mistake grants direct access as whoever the key belongs to.

## Alternatives and tradeoffs

- Manual checklist (this page): slower, but builds the understanding needed to explain a finding under interview questioning, and catches things a script's heuristics might miss or mis-rank.
- Automated enumeration scripts (LinPEAS, LinEnum): faster and more thorough in raw coverage, but risk becoming a crutch if the output is acted on without understanding why each flagged item matters. Use after the manual pass, not instead of it, especially while this checklist is still being internalized.

## Next step

**Any SUID binary or `sudo -l` entry found: go to GTFOBins before anything else.** This is the single biggest lesson from the Nibbles box, see [weak-points](../../weak-points/README.md) and the full walkthrough at [boxes/nibbles.md](../../boxes/nibbles.md).

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
