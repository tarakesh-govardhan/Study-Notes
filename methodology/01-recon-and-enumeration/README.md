# Phase 1: Recon and Enumeration

Build the full picture of the target before touching anything: hosts, ports, services, versions, technologies. Enumeration is where most later wins come from.

**Fed by modules:** 03 Network Enumeration With Nmap, 04 Footprinting, 05 Information Gathering: Web Edition, 06 Vulnerability Assessment

## Technique pages

- [Host discovery: ARP vs ICMP](host-discovery.md)
- [Port scanning fundamentals: SYN vs connect, port states](port-scanning-fundamentals.md)
- [UDP scanning](udp-scanning.md)
- [Service and version detection, manual verification](service-version-detection.md)
- [OS fingerprinting: TTL and -O](os-fingerprinting.md)
- [NSE scripting](nse-scripting.md)
- [Scan performance tuning](scan-performance-tuning.md)
- [Firewall and IDS/IPS evasion](firewall-ids-evasion.md)
- [DNS service enumeration vs reverse DNS](dns-service-enumeration.md)
- [Quick reference: common ports and first-pass service checks](quick-reference-ports-and-first-pass.md)
- [Quick reference: web enumeration, first pass](quick-reference-web-enumeration.md)

## Related weak points

- OS-ID answered by cross-box pattern-guessing instead of verification: see [weak-points](../../weak-points/README.md)
- Filtered-result tunnel vision (guessing more services instead of suspecting the scan method): see [weak-points](../../weak-points/README.md)
