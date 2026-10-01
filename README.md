# Study Notes

My working notes for the HTB Certified Penetration Testing Specialist (CPTS) path.

The repo has two layers on purpose:

| Layer | Folder | Use it when |
|---|---|---|
| Study log | [`modules/`](modules/) | I am working through an Academy module and recording what I learned |
| Lookup layer | [`methodology/`](methodology/) | I am testing a target and need the next step, fast |

Module notes feed the methodology pages. During an assessment I work from `methodology/`, organized by attack phase, not by the order I happened to study things.

## Layout

```
methodology/   technique pages grouped by phase (recon, services, web, foothold, privesc, pivoting, AD, reporting)
modules/       one folder per Academy module, plus a progress index
weak-points/   gaps I have found in my own understanding, each with a fix plan
boxes/         post-attempt notes on retired machines
templates/     page templates and a worked example
```

Start at [`modules/README.md`](modules/README.md) for progress, or [`methodology/README.md`](methodology/README.md) for the phase map.

## How every technique page is written

Each page answers: when to use it, what it needs, the command with every flag explained, what success and failure look like, and why it works. See [`templates/technique-page.md`](templates/technique-page.md). If I cannot fill in the "why it works" section, I have not learned it yet.

## Content rules

- Everything is written in my own words. No copied module text.
- No flags, passwords, or solutions for active machines. No exam content.
- No client or employer data of any kind, including sanitized versions.

## Disclaimer

All techniques are practiced in authorized lab environments and documented for educational purposes only.
