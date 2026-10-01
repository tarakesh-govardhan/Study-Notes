# OS Fingerprinting (TTL and -O)

**Phase:** 01-recon-and-enumeration
**Platform:** Network
**Source module(s):** 03 Network Enumeration With Nmap (sections 3, 10)

## When to use

Once at least one port is confirmed open, to narrow down the target OS and inform which privesc checklist and which exploit candidates apply later.

## Prerequisites

At least one port confirmed **open**. For `-O` specifically, ideally one open **and** one closed port, to give the fingerprint algorithm something to differentiate against.

## Command

```bash
# cheap heuristic, no extra flags needed, read from any response
ping -c 1 <target>

# full fingerprint
nmap -O <target>
```

| Part | What it does |
|---|---|
| (TTL value in a ping/scan response) | Approximate OS family heuristic, see table below |
| `-O` | Full OS fingerprint using Nmap's probe database |

Default TTL heuristic:

| Default TTL | Likely OS |
|---|---|
| ~128 | Windows |
| ~64 | Linux/Unix |
| ~255 | Network devices or older Unix |

## Expected output

`-O` returns a best-guess OS and version with a confidence percentage, or a list of close matches if the sample is ambiguous.

## What failure looks like

- "Too many fingerprints match" or no confident result: usually means too narrow a sample, i.e. not enough of a confirmed open/closed pair to match against.
- A confident-looking wrong answer from TTL alone: TTL decrements per network hop and can be deliberately altered, so it is a heuristic signal, not proof.

## Why it works

TTL fingerprinting works because different OS network stacks ship with different default starting TTLs, and that default survives until it has decremented past zero. It is cheap and low-noise but trivially spoofable and degraded by hop count, so it is a first guess, not a conclusion.

`-O` works by comparing a set of crafted probes and their responses against Nmap's OS fingerprint database, which needs both an open and closed port on the target to have enough signal to disambiguate between candidate OS stacks.

**The mistake to avoid: do not infer OS or service identity from a port's conventional label.** Nmap's default scan labels ports by a lookup table (for example 3389 as "ms-wbt-server"), not by a verified running service. A closed or filtered port with a Windows-sounding label is not evidence of Windows, since nothing was confirmed running there. Only reason about OS/service from ports confirmed **open**, then verify with TTL, `-sV` banners, or `-O`.

## Alternatives and tradeoffs

- TTL check: fastest, lowest noise, least reliable alone.
- `-sV` banner grab on an open port: often gives a direct OS hint (e.g. a banner string naming the distribution) without the overhead of a full `-O` fingerprint.
- `-O`: most thorough, needs more signal (an open/closed pair) and generates more traffic/noise than the other two.

## Next step

Cross-check at least two of the three methods before committing to an OS-ID answer in notes or a report. Feed the result into privesc checklist selection ([05-privesc-linux](../05-privesc-linux/README.md) or [06-privesc-windows](../06-privesc-windows/README.md)).

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
