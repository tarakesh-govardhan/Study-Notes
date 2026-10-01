# Firewall and IDS/IPS Evasion

**Phase:** 01-recon-and-enumeration
**Platform:** Network
**Source module(s):** 03 Network Enumeration With Nmap (sections 8, 9, 12)

## When to use

When a target is behind a firewall that is interfering with normal scanning (consistent `filtered` results across multiple ports or services), or when stealth against an IDS/IPS matters for the engagement.

## Prerequisites

Root for the raw packet crafting these techniques rely on. Awareness that evasion techniques are more intrusive/noisy in a different way (deliberately spoofing traffic characteristics), so use them with the same engagement-scope awareness as any other technique.

## Command

```bash
nmap -sA <target>                            # firewall rule mapping, not port state
nmap -D RND:5 <target>                       # decoys
nmap -S <spoofed-ip> -e <interface> <target> # source IP / interface
sudo nmap -g53 --max-retries=1 -Pn -p- --disable-arp-ping <target>   # source port trust exploit
```

| Part | What it does |
|---|---|
| `-sA` | ACK scan: sends ACK only |
| `-D RND:5` | 5 random decoy source IPs, real IP hidden among them |
| `-S <ip>` | Manually set the source IP |
| `-e <interface>` | Force a specific network interface, e.g. `tun0` |
| `--source-port 53` / `-g53` | Spoof the source port as 53 (DNS), to exploit implicit trust some firewalls give DNS traffic |

## Expected output

- ACK scan: `unfiltered` (RST came back, reachable) or `filtered` (no response). Never "open" or "closed" directly, see below.
- Source-port trust exploit: a port that showed `filtered` under a normal scan shows `open` when the scan is sent from source port 53.

## What failure looks like

- Decoys that do not actually hide anything: decoy IPs must be alive, or the target's SYN-flood protection may block the real service outright. ISPs and routers also often filter obviously spoofed packets, so decoys are not always effective.
- A "filtered" result interpreted as "the port is closed" is a misread: `-sA` cannot tell open from closed at all, only reachable (unfiltered) from not (filtered). Pair it with `-sS` separately for actual state.

## Why it works

**ACK scan (`-sA`) is for mapping firewall rules, not discovering port state.** Firewalls often pass ACK packets because they look like legitimate established-connection traffic rather than a new connection attempt. An RST response means the packet reached the host and bounced back, i.e. the firewall is not blocking that port's traffic class, hence "unfiltered" (reachable), not "open".

**Source port manipulation is the standout technique from this module.** DNS traffic (port 53) is often implicitly trusted by firewall rules, on the assumption that traffic from a DNS server is legitimate. Spoofing the source port as 53 exploits that trust: a port that is actually open but normally filtered to arbitrary source ports becomes reachable once the probe appears to originate from port 53. This worked directly in the hardened-target session (port 50000 went from `filtered` to `open` once scanned from source port 53), and the same trick had to be reapplied with `ncat --source-port 53` for the manual banner-grab follow-up, since the firewall rule applies to any connection, not just the initial Nmap scan.

**Detecting IDS/IPS:** IDS is passive (detects and alerts), IPS is active (blocks), and the two complement rather than duplicate each other. One detection method: scan from an isolated VPS; if that VPS's IP gets blocked afterward, IPS is active, so switch source and go quieter from there.

## Alternatives and tradeoffs

- `-sA` for firewall mapping vs `-sS`/`-sT` for actual port state: these answer different questions and are meant to be used together, not as substitutes for each other.
- Decoys: cheap to add, but unreliable and sometimes counterproductive (SYN-flood protection, ISP filtering).
- Source IP/interface spoofing (`-S`, `-e`): useful when a firewall rule is source-IP-specific, since a "filtered" result might mean the scanner's own source is blocked rather than the port itself being closed.
- Source port spoofing (`-g53`): the highest-value technique here when a target trusts DNS-sourced traffic; it is also the fix for the "consistently filtered across many services" pattern described below.

## Next step

**The pattern recognition that matters more than any single flag:** consistently `filtered` results across multiple different candidate ports/services on the same host is a signal to reconsider the scanning method itself, not a cue to keep guessing more services. See the weak point logged in [weak-points](../../weak-points/README.md) from the hardened-target session, where the actual blocker was the scan traffic being firewalled regardless of destination, and the fix was this page's source-port technique, not a better service guess.

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
