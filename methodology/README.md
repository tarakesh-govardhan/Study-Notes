# Methodology (lookup layer)

Organized by attack phase, in the order a test usually flows. Phases loop: new credentials or a new network segment send me back to recon.

| Phase | Folder | Exit criteria (when I can move on) |
|---|---|---|
| 1. Recon and enumeration | [01-recon-and-enumeration](01-recon-and-enumeration/) | Full inventory of hosts, ports, services, and versions, and every service has a next action |
| 2. Services and credentials | [02-services-and-credentials](02-services-and-credentials/) | Every exposed service checked for anonymous access, default or weak credentials, and known misconfigurations |
| 3. Web | [03-web](03-web/) | Every web entry point mapped and tested for the main vulnerability classes, with results recorded either way |
| 4. Foothold and shells | [04-foothold-and-shells](04-foothold-and-shells/) | A stable, upgraded shell and a reliable way to move files to and from the target |
| 5. Privilege escalation (Linux) | [05-privesc-linux](05-privesc-linux/) | Local enumeration complete and each finding checked, or root obtained |
| 6. Privilege escalation (Windows) | [06-privesc-windows](06-privesc-windows/) | Local enumeration complete and each finding checked, or SYSTEM/admin obtained |
| 7. Pivoting and tunneling | [07-pivoting-and-tunneling](07-pivoting-and-tunneling/) | Every reachable internal segment identified and a working path into each |
| 8. Active Directory | [08-active-directory](08-active-directory/) | Domain mapped, credentials and attack paths evaluated, impact demonstrated |
| 9. Reporting | [09-reporting](09-reporting/) | Every finding has evidence, reproduction steps, impact, and remediation |

The engagement-level process (pre-engagement through post-engagement) is in [`modules/01-penetration-testing-process`](../modules/01-penetration-testing-process/).

## Page rules

- One technique per page, using [`../templates/technique-page.md`](../templates/technique-page.md).
- Name files in kebab-case, e.g. `suid-enumeration.md`.
- Every page lists the module(s) it came from.
- Add the page to its phase README index when you create it.
