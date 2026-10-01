# Module 03: Network Enumeration With Nmap

**Phase(s):** [01-recon-and-enumeration](../../methodology/01-recon-and-enumeration/README.md)
**Completed:** CPTS path, module 3 of 28, sections 1-12. Academy's own progress bar showed 75% because I revisited some sections after finishing; the module is done.

## Summary

The core enumeration tool of the whole path. Covers Nmap's five scan categories (host discovery, port scanning, service/version detection, OS detection, NSE), how to read port states honestly instead of trusting a single scan, performance tuning, and firewall/IDS evasion. The last two sections (10 to 12) were a live, instructor-guided session against a hardened target and produced the highest-value lessons in the module: verify before concluding, and recognize when a pattern of results means the scanning method is the problem, not the target.

## Key concepts

- **Five scan categories as a planning model:** host discovery, port scanning, service/version detection, OS detection, NSE. Any engagement's enumeration can be structured around this list.
- **Open/closed/filtered is the base response-mapping logic**, reused across every scan type (SYN, ACK, UDP). Learn it once, it is not scan-type-specific.
- **Dropped vs rejected are different firewall behaviors**, not the same thing labeled "filtered": dropped means silence and retries (`--max-retries`), rejected means an explicit ICMP unreachable comes back. `--packet-trace` shows which one is happening.
- **Manual verification beats trusting a single tool's summary.** This shows up three separate times in the module: `-sV`'s banner summary vs a raw `nc` connection, `systemctl status` vs an actual `lsof`/`ss` check for what is really bound to a port, and OS-ID conclusions from pattern-guessing vs an actual TTL/`-sV`/`-O` check.
- **A result pattern is itself a signal.** Consistent `filtered` across several different candidate services on the same host is not "try more services", it is "the scan technique itself is being firewalled". This was the real unlock in section 12, not a better guess at what service was running.

## Techniques learned

Each one has its own page in the methodology lookup layer.

- [Host discovery: ARP vs ICMP](../../methodology/01-recon-and-enumeration/host-discovery.md)
- [Port scanning fundamentals: SYN vs connect, port states, dropped vs rejected](../../methodology/01-recon-and-enumeration/port-scanning-fundamentals.md)
- [UDP scanning](../../methodology/01-recon-and-enumeration/udp-scanning.md)
- [Service and version detection, manual banner verification](../../methodology/01-recon-and-enumeration/service-version-detection.md)
- [OS fingerprinting: TTL heuristic and -O](../../methodology/01-recon-and-enumeration/os-fingerprinting.md)
- [NSE scripting: categories, -sC, -A, --script vuln](../../methodology/01-recon-and-enumeration/nse-scripting.md)
- [Scan performance tuning](../../methodology/01-recon-and-enumeration/scan-performance-tuning.md)
- [Firewall and IDS/IPS evasion: ACK scan, decoys, source port spoofing](../../methodology/01-recon-and-enumeration/firewall-ids-evasion.md)
- [DNS service enumeration vs reverse DNS](../../methodology/01-recon-and-enumeration/dns-service-enumeration.md)

## Lab and exercise log

### Section 12: hardened target, full live session

Scenario: the lab administrator had retrained on IDS/IPS, relocated services, and tightened the alert-detection threshold compared to earlier boxes in the module.

Working command that succeeded after earlier approaches stalled:
```bash
sudo nmap -g53 --max-retries=1 -Pn -p- --disable-arp-ping <target>
```
Found port 50000, initially labeled `ibm-db2` by Nmap's port-number lookup (a label, not a verified service, see the OS-fingerprinting page for why that distinction matters). Follow-up version scan needed the same source-port bypass:
```bash
sudo nmap -sV -g53 --max-retries=1 -Pn -p 50000 --disable-arp-ping <target>
```
Result: `tcpwrapped`, meaning the TCP handshake succeeded but the service or a TCP-wrapper access-control layer refused to answer `-sV`'s generic probes. Manual follow-up with the same evasion technique:
```bash
ncat -nv --source-port 53 <target> 50000
```

Process lessons from this session that are not tied to one command, logged here rather than on a technique page:
- **Alert counters do not scale 1:1 with packets.** A 1000-port scan only cost about 7 alerts, which ruled out a naive per-packet counting model. Likely counts detected suspicious patterns, not raw volume. Recalibrate the mental model as data comes in instead of assuming.
- **Target resets change the IP, not the underlying box.** Academy modules generally redeploy the same vulnerable image on reset, so the answer stays consistent but the IP does not. Always verify the current live target IP from the lab panel before scanning.
- **Timing and port count multiply, they do not add.** `-p- -T2` on a large range can balloon to many hours. Early ETC estimates are unstable and often correct downward, so do not panic-cancel immediately, but do reconsider the approach if the combination was clearly miscalibrated.
- **Top-1000 (the default scan with no port flag) is a real middle ground** between top-100 and `-p-`. Worth trying before committing to a full range scan.
- **SFTP has no separate port.** It rides over SSH (port 22) as a subsystem. If SSH is open, check `-sV` on 22 itself rather than hunting for a dedicated SFTP port.
- **New bulk-transfer protocol vocabulary this session:** TFTP (UDP 69, no auth, firmware/config transfer), rsync (TCP 873, purpose-built for efficient large/delta transfers), NFS (2049, plus 111 rpcbind/portmapper), SMB/CIFS (445).
- **Tooling gotcha:** traditional `nc` uses `-p <port>` for source port, `ncat` (the Nmap-project version) supports `--source-port <port>`. Check which is installed with `which ncat` before assuming the syntax.
- **Privileged port bind conflicts:** ports below 1024 need root. "Address already in use" on port 53 specifically often means a local DNS resolver is already bound there. `systemctl status dnsmasq` showing inactive does not guarantee the port is free; a separate dnsmasq instance (for example a libvirt/VM-bridge network's own instance) can be running independently of the systemd-managed service of the same name. Verify with a fresh `lsof -i :53` or `ss -tulnp | grep :53`, identify the actual PID, and kill it directly if confirmed safe.

## Where I got stuck

Two gaps surfaced in this module, both now tracked in [weak-points](../../weak-points/README.md):

1. **OS-ID answered from cross-box pattern-guessing, not verification.** Got the correct answer (the target OS) from recognizing a pattern across earlier labs rather than actually checking TTL, `-sV`, or `-O` on this specific target before answering.
2. **Filtered-result tunnel vision.** Spent significant time and alert budget cycling through service guesses (FTP, SFTP, TFTP, rsync, NFS) under repeated `filtered` results before recognizing that consistent filtering across every candidate pointed to a scan-technique problem, not a target-selection problem.

## Open questions

- Have not yet tried `dig @<target> version.bind chaos txt` for BIND-specific version fingerprinting beyond what Nmap's `-sV` gives. Worth trying next time a DNS service shows up.
