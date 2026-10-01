# NSE Scripting (Nmap Scripting Engine)

**Phase:** 01-recon-and-enumeration
**Platform:** Network
**Source module(s):** 03 Network Enumeration With Nmap (section 6)

## When to use

Once a service is identified, to either run broad default checks or target a specific thing, such as "find exposed content" or "check this service for known vulnerabilities".

## Prerequisites

A target port where a relevant NSE script category applies (knowing the service helps pick the right category instead of guessing).

## Command

```bash
nmap -sC -sV <target>                     # default scripts
nmap --script discovery -p 80 <target>    # a whole category
nmap --script http-enum -p 80 <target>    # a specific script
nmap -A <target>                          # heavier, all-in-one
```

| Part | What it does |
|---|---|
| `-sC` | Runs the `default` script category |
| `--script <category>` | Runs every script in that category |
| `--script <name>,<name>` | Runs specific named scripts |
| `--script "http-*"` | Wildcard match on the name prefix across **all** categories, not a category filter |
| `-A` | Shorthand for `-sV` + `-O` + `--traceroute` + `-sC` combined |

## Expected output

Script output appears nested under the relevant port in the scan results, varying by script (a table of discovered paths for `http-enum`, CVE references for `vulners`, etc).

## What failure looks like

A wildcard script run (`--script "http-*"`) that takes far longer than expected and includes brute-force or fuzzing scripts was not intended: the wildcard matches every category sharing that name prefix, including `brute`, `dos`, and `fuzzer`, not just the safe/discovery ones. This is a scope mistake, not a tool bug.

## Why it works

NSE scripts are organized into 14 categories, each with a different risk/purpose profile:

| Category | Purpose |
|---|---|
| auth | Credential determination |
| broadcast | Host discovery via broadcast |
| brute | Login brute-forcing |
| default | Runs with `-sC` |
| discovery | Evaluates accessible services, content/service enumeration |
| dos | DoS vulnerability checks (rarely used) |
| exploit | Attempts known exploits |
| external | Uses external services |
| fuzzer | Malformed input, slow by design |
| intrusive | Can negatively affect the target |
| malware | Checks for malware infection |
| safe | Non-destructive |
| version | Service detection extension |
| vuln | Identifies specific vulnerabilities |

Category choice matters: `discovery`/`http-enum` is the right tool for "find exposed content on a web service", not a wildcard or `-A`. This directly found a real result in `robots.txt` during the module.

## Alternatives and tradeoffs

- `-sC`: safe default, good first pass, limited depth.
- `--script <category>`: targeted and predictable in scope.
- `--script "name-*"`: convenient but dangerously broad, crosses category boundaries silently.
- `-A`: thorough but heavy; use once a port is already worth deep-diving, not as a first move on every port.

## Next step

Follow category-specific findings into their relevant phase: `discovery`/`http-enum` results feed [03-web](../03-web/README.md), `vuln` results feed exploit search.

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
