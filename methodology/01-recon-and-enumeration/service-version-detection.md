# Service and Version Detection, Manual Verification

**Phase:** 01-recon-and-enumeration
**Platform:** Network
**Source module(s):** 03 Network Enumeration With Nmap (sections 5, 12)

## When to use

On every port confirmed open, after the fast full-range sweep, to identify exactly what is running and its version.

## Prerequisites

A list of confirmed open ports (from the two-stage methodology in [port-scanning-fundamentals](port-scanning-fundamentals.md)).

## Command

```bash
nmap -sV -p <open-ports> <target>
nc -nv <target> <port>
```

| Part | What it does |
|---|---|
| `-sV` | Layers protocol-specific probes on the given ports to identify service and version |
| `-p <open-ports>` | Scope the version scan to only the already-confirmed open ports, since `-sV` cost scales with ports probed, not the range scanned |
| `nc -nv <target> <port>` | Raw banner grab: connect directly and read whatever the service sends unprompted |

## Expected output

`-sV` output names a service and, where detectable, a version string, e.g. `25/tcp open smtp Postfix smtpd`.

## What failure looks like

- `tcpwrapped`: the TCP handshake succeeded but the service (or a TCP-wrapper access-control layer) refused to answer `-sV`'s generic probes. Not a scan failure, it is itself a signal of a hardened or access-controlled service.
- A banner that contradicts or adds detail beyond the `-sV` summary: `-sV` summarized a service as `Postfix smtpd`, but the raw banner itself also named the underlying distribution. `-sV` can miss detail present in the raw banner.

## Why it works

`-sV` sends a sequence of protocol probes and matches the responses against a signature database. It is fast and automated, but it is still a guess based on pattern matching, so it can miss or mis-summarize detail that a direct look at the raw banner would show.

A manual `nc` connection bypasses that summarization entirely: whatever the service sends on connect is read as-is. Pairing it with `tcpdump -i eth0 host <your_ip> and <target_ip>` shows the exchange at the packet level: SYN, SYN-ACK, ACK, then a PSH-ACK carrying the banner, then ACK. The PSH flag marks the server actively pushing data rather than just acknowledging.

**The habit this reinforces: do not fully trust `-sV`'s summary. A manual `nc` connection is a good verification reflex, not just a fallback for when `-sV` fails outright.**

## Alternatives and tradeoffs

- `-sV` alone: fast, automated, good enough for most ports, but a summary can hide detail.
- `nc`/`ncat` manual connection: slower (one port at a time, by hand), but ground truth for what the service actually says.
- `ncat --source-port 53 <target> <port>`: needed when the target is filtering by source port and a plain `nc`/`ncat` connection would be dropped before any banner is seen; see [firewall-ids-evasion](firewall-ids-evasion.md).

## Next step

Feed the confirmed service and version into exploit search (`searchsploit <app> <version>`) and into OS fingerprinting cross-checks, see [os-fingerprinting](os-fingerprinting.md).

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
