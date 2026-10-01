# Quick Reference: Common Ports and First-Pass Service Checks

**Phase:** 01-recon-and-enumeration
**Platform:** Network / Linux / Windows
**Source module(s):** 02 Getting Started

## When to use

Early in an engagement, as a fast first pass across commonly-seen services before going deep on any one of them. This is a reference page rather than a single technique; link out from here to the dedicated pages once a service is confirmed open.

## Common ports

| Port | Service |
|---|---|
| 20/21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 80 | HTTP |
| 161 | SNMP |
| 389 | LDAP |
| 443 | HTTPS |
| 445 | SMB |
| 3389 | RDP |

## First-pass commands

```bash
nmap <ip>                      # top 1000 ports, quick
nmap -sV -sC -p- <ip>          # all ports, version detection, default scripts
```

Cross-reference service version to OS version where possible, for example an OpenSSH package version string mapping to a specific Ubuntu release.

**FTP:**
```bash
ftp <ip>
# try anonymous login, then:
ls / cd / get <file>           # check downloaded files for credentials
```

**SMB:**
```bash
nmap --script smb-os-discovery.nse -p445 <ip>
smbclient -N -L \\<ip>                      # list shares, null session
smbclient -U <user> \\<ip>\<share>          # connect to a share
```

**SNMP:**
```bash
snmpwalk -v 2c -c public <ip> <oid>         # try default community strings: public / private
```

## Why these are grouped here

These are first-pass, low-depth checks meant to quickly surface a foothold or an obvious misconfiguration (anonymous FTP, null SMB session, default SNMP community) before committing time to deeper enumeration elsewhere. If one of these pays off immediately, it can shortcut the rest of the recon phase.

## Next step

If a service here yields credentials or a shell directly, go straight to [04-foothold-and-shells](../04-foothold-and-shells/README.md). If it yields only information, fold that into the broader picture built by the rest of [01-recon-and-enumeration](README.md).
