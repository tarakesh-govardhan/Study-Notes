# Host Discovery

**Phase:** 01-recon-and-enumeration
**Platform:** Network
**Source module(s):** 03 Network Enumeration With Nmap (sections 1-3)

## When to use

Start of any engagement against a network range: find which hosts are actually live before spending time port-scanning dead IPs.

## Prerequisites

Network reachability to the target range. Root (or equivalent) for raw ARP/ICMP crafting.

## Command

```bash
nmap -sn <target-range>
nmap -sn -oA hostdiscovery -iL hosts.lst
```

| Part | What it does |
|---|---|
| `-sn` | Disable port scan, host discovery only |
| `-oA <name>` | Save output in all three formats (`.nmap`, `.gnmap`, `.xml`) |
| `-iL hosts.lst` | Read targets from a file instead of the command line |
| `10.129.2.18-20` | Range shorthand for consecutive IPs; space-separated IPs also works for a non-contiguous list |

## Expected output

A list of hosts that responded, each marked "Host is up".

## What failure looks like

A host reported down that is actually up and simply not responding to the discovery method used (common behind a firewall). Do not trust a single negative result before checking the method, see below.

## Why it works

**On the local subnet, `-sn` defaults to ARP request/reply, not ICMP**, even though ICMP (ping) is the intuitive assumption. ARP is used because it works at layer 2 and is rarely filtered on a local segment, so it is more reliable than ICMP for this purpose.

```bash
nmap -sn -PE --disable-arp-ping <target>
```
- `-PE` forces an ICMP echo request specifically.
- `--disable-arp-ping` is required alongside `-PE` for the ARP fallback not to take over again; without it, ARP still wins on a local subnet regardless of `-PE`.
- `--packet-trace` shows the raw packets sent and received, the diagnostic reflex for "what is actually happening on the wire".
- `--reason` shows why Nmap concluded a given state, for example "received arp-response".

The general lesson: do not assume what discovery method is in play. Verify with `--packet-trace` or `--reason` instead of assuming `-sn` always means ICMP.

## Alternatives and tradeoffs

- ARP (default, local subnet): fast, hard to filter, but only works on the same layer-2 segment.
- ICMP (`-PE`, forced): works across subnets/routed networks, but is commonly filtered by firewalls, producing false negatives.
- Neither is strictly better; the choice depends on whether the target is on the same segment and whether ICMP is likely filtered.

## Next step

For anything reported up, move to port scanning. For anything reported down on a routed target, consider that ICMP may simply be filtered rather than the host being offline, and re-check with a different discovery method or `-Pn` later if a specific port is known to be open from another source.

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
