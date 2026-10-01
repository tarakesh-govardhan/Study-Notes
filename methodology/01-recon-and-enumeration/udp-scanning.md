# UDP Scanning

**Phase:** 01-recon-and-enumeration
**Platform:** Network
**Source module(s):** 03 Network Enumeration With Nmap (section 4), reinforced in section 11 (DNS)

## When to use

Whenever a UDP-based service is suspected or confirmed in scope: DNS, SNMP, TFTP, NFS portmapper, and similar. UDP services rarely show up in a TCP-only sweep and will not appear in low top-N port lists reliably.

## Prerequisites

Root for raw packet crafting. Patience: UDP scanning is inherently much slower than TCP.

## Command

```bash
sudo nmap -sU -sV -T2 -p 53 <target>
```

| Part | What it does |
|---|---|
| `-sU` | UDP scan |
| `-sV` | Version detection layered on top |
| `-T2` | Slower timing template, appropriate given UDP's own slowness and false-positive risk at high speed |
| `-p 53` | Target the specific port rather than scanning the full UDP range, which is extremely slow |

## Expected output

- A response from the service: confirmed **open**.
- An ICMP port-unreachable response: confirmed **closed**.
- No response at all: **open\|filtered**, ambiguous, the most common UDP result.

## What failure looks like

A scan that appears to hang or takes an extremely long time on a large UDP port range. This is expected behavior, not a bug: UDP is stateless, there is no handshake to shortcut the state machine, so retries and timeouts dominate. Scope the port list down instead of troubleshooting the hang.

## Why it works

Unlike TCP's deterministic SYN/SYN-ACK/RST exchange, UDP has no built-in connection setup, so Nmap cannot distinguish "nothing is listening" from "something is listening but dropped or ignored the packet" unless the service or an ICMP error explicitly responds. That ambiguity, `open|filtered`, is the normal result for most UDP ports and is not a failure of the scan.

## Alternatives and tradeoffs

- Scanning the full UDP range is rarely worth the time cost; scope to specific suspected ports instead (DNS 53, SNMP 161, TFTP 69, NFS 2049/111).
- A top-N port list may not include a UDP service even if it is open, since top-N lists are based on global common-port frequency, not guaranteed inclusion. Check explicitly rather than trusting a top-100/1000 scan to have covered it.

## Next step

For any UDP port confirmed open, follow with a protocol-specific manual check where one exists, see [dns-service-enumeration](dns-service-enumeration.md) for the DNS case.

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
