# Module 02: Getting Started

**Phase(s):** Foundations, feeds 01 recon, 02 services and credentials, 03 web, 04 foothold and shells, 05 privesc linux
**Completed:** CPTS path, module 2 of 28 (23 sections, includes the Nibbles box)

## Summary

The toolbox module: organization habits, the core tools (SSH, netcat, tmux, vim), a first pass at service scanning and web enumeration, shell types and the TTY upgrade sequence, a privilege escalation checklist, file transfer methods, and VPN/Burp troubleshooting. Ends with the Nibbles box as a full first attempt. Most of the content here is reference material that belongs in the lookup layer rather than prose, so this page stays short and links out to the technique pages.

## Key concepts

### Staying organized

Folder structure per engagement or box:
```
Projects/[Client or Box Name]/
├── evidence (credentials, data, screenshots)
├── logs
├── scans
├── scope
└── tools
```
Maintain a running knowledge base: cheat sheets, checklists, findings templates. Aggregate every payload or command seen, since any of it may get reused. This repo is that knowledge base.

### Why SSH over a reverse shell when both are available

SSH is more stable and can be used as a jump host. Worth remembering as a reason to prefer a legitimate credential-based foothold over popping a reverse shell when both paths exist.

### Shell types

| Type | Behavior |
|---|---|
| Reverse | Target connects back to me |
| Bind | I connect to the target's listening port |
| Web | Commands sent via HTTP params (PHP/JSP/ASP) |

### TTY upgrade sequence

Standard sequence I now have memorized:
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z
stty raw -echo
fg
# Enter twice
export TERM=xterm-256color
stty rows <X> columns <Y>
```
If `python` is not found, try `python3`. Many modern systems do not alias `python` to Python 2 by default.

### Privilege escalation checklist (first pass, pre-dedicated modules)

- Kernel exploits (OS/kernel version vs known CVEs)
- Vulnerable installed software
- `sudo -l`, check for `NOPASSWD` entries, then look up the binary in GTFOBins
- SUID binaries, same GTFOBins lookup
- Cron jobs and scheduled tasks: writable job files or directories
- Exposed creds in config files, logs, bash_history
- Readable SSH keys (`id_rsa`): copy, `chmod 600`, `ssh -i key`

## Techniques learned

These moved into the methodology lookup layer rather than staying here:

- Common ports quick reference, service scanning first pass, FTP/SMB/SNMP workflows: see [methodology/01-recon-and-enumeration](../../methodology/01-recon-and-enumeration/README.md)
- Web enumeration basics (gobuster, curl, whatweb, robots.txt, page source, SSL SAN leakage): see [methodology/01-recon-and-enumeration](../../methodology/01-recon-and-enumeration/README.md)
- Reverse/bind/web shells, netcat listener, Linux reverse shell one-liner, mkfifo alternative, TTY upgrade, web shell payload: see [methodology/04-foothold-and-shells](../../methodology/04-foothold-and-shells/README.md)
- File transfer (HTTP server, SCP, base64 for restricted environments, integrity checks): see [methodology/04-foothold-and-shells](../../methodology/04-foothold-and-shells/README.md)
- Privilege escalation checklist and GTFOBins lookup habit: see [methodology/05-privesc-linux](../../methodology/05-privesc-linux/README.md)
- Public exploit search and Metasploit flow: see [methodology/02-services-and-credentials](../../methodology/02-services-and-credentials/README.md)

## Lab and exercise log

### Nibbles box (first box completed in the path, both flags on first attempt)

Full walkthrough: [boxes/nibbles.md](../../boxes/nibbles.md)

## Where I got stuck

- Had never used GTFOBins before the Nibbles box. Logged in detail under the box notes and tracked in [weak-points](../../weak-points/README.md).

## Open questions

None outstanding for this module.
