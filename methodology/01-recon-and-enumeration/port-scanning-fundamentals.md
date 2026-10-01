# Port Scanning Fundamentals (SYN vs Connect, Port States)

**Phase:** 01-recon-and-enumeration
**Platform:** Network
**Source module(s):** 03 Network Enumeration With Nmap (sections 1, 4)

## When to use

After host discovery confirms a target is live, to find which services are reachable on it.

## Prerequisites

Root/raw-socket access for `-sS` (SYN scan). Without it, Nmap falls back to `-sT` automatically.

## Command

```bash
sudo nmap -sS -p- <target>     # full range, SYN scan
nmap -sT -p 1-1000 <target>    # connect scan, no root needed
```

| Part | What it does |
|---|---|
| `-sS` | SYN scan: sends SYN only, never completes the handshake |
| `-sT` | Connect scan: completes the full TCP handshake |
| `-p-` | All 65535 ports |
| `-p 22,25,80` | Specific port list |
| `-p 22-445` | Port range |
| `--top-ports=10` / `-F` | Top N most common ports (10, or top 100 with `-F`) |

## Expected output

Each scanned port reported as one of 6 states:

| State | Meaning |
|---|---|
| open | Connection established |
| closed | RST received |
| filtered | No response or an error, firewall likely |
| unfiltered | ACK scan only; reachable but open/closed undetermined |
| open\|filtered | No response, cannot distinguish (common on UDP) |
| closed\|filtered | Idle scans only, cannot distinguish |

## What failure looks like

A scan that takes far longer than expected on a large port range usually means retries are stacking up on filtered/dropped ports (see `--max-retries` under [scan-performance-tuning](scan-performance-tuning.md)), not that something is broken.

## Why it works

**SYN scan logic:** sends a SYN packet and reads the response without completing the handshake.

| Response | Port state |
|---|---|
| SYN-ACK | open |
| RST | closed |
| no response | filtered (likely firewall drop) |

This is "half-open": since the handshake never completes, it is faster and stealthier than a full connect, though modern IDS/IPS often still catches it.

**Connect scan (`-sT`):** completes the full three-way handshake, which is why it does not need raw-socket privileges (it uses the OS's normal socket API). It behaves exactly like a normal client connecting, so it is more "polite" but fully logged on the target, i.e. less stealthy.

**Filtered is not one thing.** It splits into two distinct firewall behaviors:
- **Dropped:** no response at all. Nmap retries up to `--max-retries` (default 10), which is why a scan against a dropping firewall takes noticeably longer.
- **Rejected:** an explicit ICMP type 3 (destination unreachable) response comes back, for example code 3 "port unreachable", or others like "net prohibited" or "host prohibited".

`--packet-trace` reveals which case is actually happening rather than guessing from the word "filtered" alone.

## Alternatives and tradeoffs

- `-sS` vs `-sT`: stealth and speed vs not needing root and leaving a less IDS-triggering footprint in some environments. Neither is categorically "better"; choose based on privilege level and how much the engagement cares about stealth.
- Full range (`-p-`) vs top-N (`-F`, `--top-ports`): full range is thorough but slow; top-N is a fast first pass. See the two-stage methodology below.

## Next step

**Two-stage scan methodology (the habit to default to):**
1. Fast full-range port scan first (`-p-`, no `-sV`), to get the open-port list with a low footprint.
2. Targeted `-sV`/NSE only on the ports that came back open from step 1, since `-sV` cost scales with ports probed, not the range scanned.

Always save with `-oA <name>` (`.nmap`, `.gnmap`, `.xml`; `xsltproc target.xml -o target.html` turns the XML into an HTML report). During a long scan, `[Space Bar]` gives live progress, `--stats-every=5s` gives periodic updates, and `-v`/`-vv` shows open ports as they are discovered rather than waiting for the full scan to finish.

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
