# SUID Binary Enumeration (Linux)

> **This is a worked example written to show how the template is filled in. Replace it with a page in my own words once I have covered the topic.**

**Phase:** 05-privesc-linux
**Platform:** Linux
**Source module(s):** 25 Linux Privilege Escalation

## When to use

I have a shell as a low-privileged user and I am running local enumeration for a way to become root.

## Prerequisites

A shell on the target. No special privileges required.

## Command

```bash
find / -perm -4000 -type f 2>/dev/null
```

| Part | What it does |
|---|---|
| `/` | Start the search at the filesystem root |
| `-perm -4000` | Match files that have the setuid bit set (the leading `-` means all listed bits must be present) |
| `-type f` | Regular files only, skipping directories |
| `2>/dev/null` | Discard stderr so "Permission denied" noise does not bury the results |

## Expected output

A list of binaries. Many are standard and expected, for example `passwd` and `sudo`. The interesting ones are anything non-standard or unusual for the distribution.

## What failure looks like

- A wall of "Permission denied" lines means `2>/dev/null` was left off.
- Only standard binaries is a valid result, not a failure. It means this vector is not available, so move on and record that.

## Why it works

A setuid binary runs with the effective user ID of its owner, not the user who launched it. If the owner is root and the binary can be made to do something attacker-controlled (spawn a shell, read or write arbitrary files, run a command), that action happens as root.

## Alternatives and tradeoffs

- Enumeration scripts such as LinPEAS report SUID files with highlighting, which is faster but noisier and hides the mechanics.
- Also check SGID (`-perm -2000`) and file capabilities (`getcap -r / 2>/dev/null`) in the same pass.

## Next step

Compare every non-standard result against GTFOBins. Check whether the binary is exploitable as SUID specifically, not just listed.

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
