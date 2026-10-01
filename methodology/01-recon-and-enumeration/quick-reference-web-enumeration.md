# Quick Reference: Web Enumeration (First Pass)

**Phase:** 01-recon-and-enumeration
**Platform:** Web
**Source module(s):** 02 Getting Started

## When to use

First pass against any web service, before deeper per-vulnerability-class testing covered in [03-web](../03-web/README.md).

## Commands

```bash
gobuster dir -u <url> -w <wordlist>         # directory brute-force
gobuster dns -d <domain> -w <wordlist>      # subdomain enum
curl -IL <url>                              # headers
whatweb <ip>                                # fingerprinting
```

HTTP status codes worth remembering while reading gobuster output: `200` ok, `403` forbidden, `301` redirect.

**Always check manually, not just with automated tools:**
- `robots.txt`
- Page source (`Ctrl+U`): comments, hints, dev leftovers
- SSL certificate details: SAN fields can leak internal IPs or hostnames

## Why this matters

The manual checks catch things automated directory brute-forcing will not: an HTML comment, a disclosed version number in a README, or an internal IP leaked through a certificate's SAN field are all things a wordlist-based scan does not look for. See the Nibbles box notes ([boxes/nibbles.md](../../boxes/nibbles.md)) for a concrete case where an HTML comment and a README version disclosure were the actual way in, not the directory brute-force itself.

## Next step

Anything found here that looks like a vulnerability class (injection, upload, inclusion, etc.) moves to [03-web](../03-web/README.md) for the dedicated technique.
