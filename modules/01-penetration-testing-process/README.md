# Module 01: Penetration Testing Process

**Phase(s):** Foundations (applies to all phases)
**Completed:** CPTS path, module 1 of 28

## Summary

The engagement-level process a pentest follows end to end: 8 phases from pre-engagement paperwork through post-engagement cleanup. The loop that matters most day to day is information gathering, vulnerability assessment, and exploitation cycling back on each other, not a straight line.

![Process overview](pt-process.png)

## The 8 phases

```
Pre-Engagement -> Information Gathering -> Vulnerability Assessment ->
Exploitation -> Post-Exploitation -> Lateral Movement -> PoC -> Post-Engagement
```

## Key concepts

### 1. Pre-Engagement

Flow: NDA, scoping questionnaire, pre-engagement meeting, contract/RoE signed, kick-off meeting.

- **RoE (Rules of Engagement):** scope, legal boundaries, contacts, comms channels. Defines what is authorized.
- **Test types:**

| Engagement | Description |
|---|---|
| Black-box | Minimal or no prior knowledge. Most realistic simulation, most time on recon. |
| Grey-box | Some info given (IP ranges, low-level creds, diagrams). Less recon, more time on exploitation. |
| White-box | Full access (admin creds, source code). Most comprehensive, finds logic flaws others miss. |

### 2. Information Gathering

Four categories: OSINT, infrastructure enumeration, service enumeration, host enumeration.

- Not a one-time phase. Repeats constantly, especially internally and post-exploitation, because each new vantage point reveals new things.
- **Pillaging:** looting data from an already-compromised host (creds, configs, files) to feed privesc and lateral movement. No dedicated Academy module for this; it is embedded across Network Enumeration with Nmap, Getting Started, Password Attacks, AD Enum & Attacks, Linux/Windows PrivEsc, and Attacking Common Services/Apps.

### 3. Vulnerability Assessment

Core skill: form a hypothesis from partial info, then test it. Example: an unusual port number suggests a service by pattern (2121 likely FTP renamed), then confirm by connecting.

- Loops constantly with information gathering, not linear.
- **Real pentest vs CTF:** thoroughness beats speed. A real test prioritizes finding everything, not the fastest path to root.

### 4. Exploitation

Prioritize attacks by: probability of success, complexity, probability of damage.

- Prepare exploits safely; test locally or on a mirrored system before running on a live target if uncertain.
- When in doubt, communicate with the client before risky exploitation. Extra communication beats an uncontrolled incident.

### 5. Post-Exploitation

Components: evasive testing, info gathering again (from inside), pillaging, vuln assessment again, privilege escalation, persistence, data exfiltration.

- Goal: sensitive data plus escalation to the highest privilege (root, Domain Admin, SYSTEM).
- Persistence is often set up early, to avoid re-exploiting an unstable vector if the connection drops.

### 6. Lateral Movement

- Goal: test how far an attacker could move through the whole network, not just one host.
- Pivoting/tunneling: using a compromised host as a proxy to reach otherwise-unreachable internal systems.
- Same loop repeats per new host: info gathering, vuln assessment, exploitation, post-exploitation.

### 7. Proof-of-Concept (PoC)

- Proves a vuln is real and exploitable, so developers/admins can reproduce and test their fix.
- **Mental model: fix the root cause, not the symptom.** A weak password is not the real problem; the weak password policy is. Fixing one account's password does not fix the systemic issue.
- Attack chain documentation shows how flaws combine. Fixing one link does not break the whole chain if other flaws remain.

### 8. Post-Engagement

Flow: cleanup (remove tools/scripts, revert changes), report (draft, client review, final), post-remediation testing, data retention/destruction, close out.

- The pentester gives general remediation advice but does not implement fixes, to stay independent and avoid a conflict of interest.

## Practice methodology

Theory alone is not enough. Structure repetition, and treat documentation practice as part of the skill, not separate from it.

Suggested ratio once comfortable with a topic: **2x modules -> 3x retired machines -> 5x active machines -> 1x Pro Lab/Endgame**.

**Module approach:** read, do the practice exercises, complete it, redo the exercises from scratch while taking notes, then turn notes into technical and non-technical documentation.

**Retired machine approach (the one I actually use):**
1. Get user flag solo
2. Get root flag solo
3. Write technical documentation
4. Write non-technical documentation
5. Compare against the official or community write-up
6. List what was missed, against the reference
7. Watch an Ippsec walkthrough, compare again
8. Expand notes with the missed parts

**Active machine approach (black-box, no write-ups exist):** get flags, write technical doc, write non-technical doc, get both proofread by a technical and a non-technical reader.

**Pro Lab/Endgame:** multi-host. Document the full attack chain (foothold to full compromise), not single-host writeups.

## Where I got stuck

Step 6 of the retired-machine approach, explicitly diffing against the reference solution and listing what was missed, is a discipline I was not doing. Folded into the weak-points tracking system: see [weak-points](../../weak-points/README.md).

## Open questions

None outstanding for this module.
