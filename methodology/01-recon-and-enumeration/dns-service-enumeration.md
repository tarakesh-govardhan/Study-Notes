# DNS Service Enumeration vs Reverse DNS

**Phase:** 01-recon-and-enumeration
**Platform:** Network
**Source module(s):** 03 Network Enumeration With Nmap (section 11)

## When to use

When the target may be running an actual DNS server (not just appearing in reverse-DNS lookups done against it), and the version of that DNS server matters for vulnerability research.

## Prerequisites

Confirming port 53 is actually open on the target. Note that it may not appear in a top-10 or even top-100 port list despite being open.

## Command

```bash
sudo nmap -sU -sV -T2 -n -Pn -p 53 <target>
dig @<target> version.bind chaos txt
```

| Part | What it does |
|---|---|
| `-sU` | DNS primarily uses UDP for queries; scan UDP, not TCP, by default |
| `-sV` | Identify the DNS server software and version on the confirmed-open port |
| `-n` | Skip Nmap's own reverse-DNS resolution of the target, not related to finding the target's own DNS service |
| `-Pn` | Skip host discovery (assume up), useful once the target is already known to be live |
| `-p 53` | Target the specific port explicitly, since 53 may not be in a default top-N list |
| `dig @<target> version.bind chaos txt` | BIND-specific version query over the CHAOS class, a separate technique from Nmap's own `-sV`, not yet tried personally |

## Expected output

`-sV` returns a service name and version string for the DNS daemon if one is confirmed open and willing to answer. `dig ... version.bind` returns a version string directly from BIND-family servers that support the CHAOS class query.

## What failure looks like

Confusing "I got a hostname back from a reverse lookup" with "I found a DNS service on the target" is the main failure mode here, not a command error. See below.

## Why it works

**Reverse DNS (PTR records, Nmap's `-n` flag) is not the same thing as a DNS server's version.** Reverse DNS resolves "what hostname maps to this IP", which is independent of ports and does not require any specific port to be open on the target being looked up. It answers a naming question, not a "is this host running a DNS service" question.

**To actually find a DNS server's version, port 53 must be open on the target itself**, then version-scanned. Disabling `-n` (Nmap's own reverse-resolution behavior) has nothing to do with this; it only changes whether Nmap resolves IPs to names for its own display purposes.

**DNS protocol note:** DNS queries are primarily UDP; TCP is used historically for zone transfers and responses too large for a single UDP packet (over 512 bytes), with TCP use increasing due to DNSSEC and IPv6 making larger responses more common.

## Alternatives and tradeoffs

- Nmap `-sV`: general-purpose, works across DNS server implementations, less detailed than a protocol-specific query.
- `dig version.bind`: BIND-specific, more detail when it works, does nothing against non-BIND DNS servers or ones that disable the CHAOS class query.

## Next step

Feed the confirmed DNS version into exploit search the same as any other service. If reverse DNS lookups were the only thing done so far, treat port 53's actual open/closed state as still unverified until explicitly checked.

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
