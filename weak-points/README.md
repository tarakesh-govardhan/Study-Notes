# Weak Points

Specific gaps in my own understanding, each with a fix plan. The rule: name the exact gap (not "bad at shells"), add one or two concrete action items, then move on. Revisit only after the action items are done.

| Gap | Phase | Action items | Status |
|---|---|---|---|
| Shell quoting across layers (single vs double quotes through shell, PHP or Python, then shell) | 04 foothold | Write three nested-quoting examples by hand and predict the final string before running each | Open |
| File descriptor redirection (`2>&1`, `exec 3<>/dev/tcp/...`, `<&3 >&3 2>&3`) | 04 foothold | Build a reverse shell from scratch on a lab VM and explain each descriptor | Open |
| Never having used GTFOBins before needing it live (Nibbles box) | 05 privesc linux | 1) Spend ~20 min browsing gtfobins.github.io directly, work one example manually without hints. 2) Skim PayloadAllTheThings' PHP section for reading fluency | In progress |
| Syntax fluency: choosing between cheat-sheet variants instead of pasting several | All | For each technique page, write the command from memory first, then compare | Open |
| OS-ID answered by cross-box pattern-guessing instead of verification on the specific target | 01 recon and enumeration | Before submitting any OS-ID answer, run at least one of TTL check, `-sV` banner, or `-O`, even when confident | Open |
| Filtered-result tunnel vision: cycling through service guesses under repeated `filtered` results instead of suspecting the scan method | 01 recon and enumeration | When multiple different candidate services on one host all show `filtered`, treat that as a signal to try firewall-evasion techniques (source port spoofing, decoys) before guessing more services. See [firewall-ids-evasion](../methodology/01-recon-and-enumeration/firewall-ids-evasion.md) | Open |
| Not explicitly diffing against the official write-up after a retired box (step 6 of the practice methodology) | All | After the next retired box, write a "what I missed" section before watching any walkthrough, compared only against the official write-up first | Open |

When a gap is closed, move its row to a "Closed" section with the date and what proved it.

Per-gap worked examples go in this folder as `<gap-name>.md`, written by me, runnable without notes.
