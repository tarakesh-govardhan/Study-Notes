# Scan Performance Tuning

**Phase:** 01-recon-and-enumeration
**Platform:** Network
**Source module(s):** 03 Network Enumeration With Nmap (section 7, reinforced in section 12)

## When to use

Any time a scan is taking too long, or conversely when speed needs to be deliberately traded for stealth against a monitored target.

## Prerequisites

None beyond a running scan or one about to be run.

## Command

```bash
nmap -T4 --max-retries=1 -p- <target>
```

| Part | What it does |
|---|---|
| `-T <0-5>` | Timing template. Default is `-T3`. 0=paranoid, 1=sneaky, 2=polite, 3=normal, 4=aggressive, 5=insane |
| `--max-retries <n>` | Retry count for ambiguous/no-response ports, default 10 |
| `--min-rate <n>` | Forces a minimum packets-per-second rate |
| `--min-parallelism <n>` | Minimum concurrent probes |
| `--initial-rtt-timeout` / `--max-rtt-timeout` | Response wait time before giving up on a probe, default around 100ms |

## Expected output

A scan that completes in a predictable, bounded time appropriate to the port count and timing template chosen.

## What failure looks like

- An ETC (estimated time to completion) that balloons into many hours: usually `-p- -T2` or similar, where a large port range and a slow timing template multiply rather than add. Early ETC estimates are unstable and often correct downward as the scan progresses, so do not panic-cancel immediately, but do reconsider if the combination was clearly miscalibrated from the start.
- A scan that is fast but misses hosts or ports that a slower scan would have found: happens when timeouts or retries are cut too aggressively for the actual network conditions.

## Why it works

Speed vs accuracy is not free. Concretely demonstrated in the module: tightening the RTT timeout missed 2 hosts, and `--max-retries 0` missed 2 open ports that a default-retry scan caught. In one specific case `--min-rate`/`-T5` happened to give the same result as a slower scan, but that is not guaranteed in general; a faster rate always trades away some margin for packets that are slow to respond or get dropped and retried.

`--max-retries` interacts directly with the dropped-vs-rejected distinction from [port-scanning-fundamentals](port-scanning-fundamentals.md): a dropped (silently firewalled) port causes Nmap to retry up to the max-retries count before giving up, which is why lowering it from the default of 10 was the single biggest speed fix for a large scan against a filtering target.

## Alternatives and tradeoffs

- **White-box or known-safe environment:** push `--min-rate` and `-T4`/`-T5` freely, since there is no need to stay quiet and the network is known to handle the load.
- **Black-box or an unfamiliar/untrusted network:** rely on the standard timing templates (`-T0` to `-T3`) rather than hand-tuning individual rate/retry flags, since hand-tuned values risk missing results on a network whose behavior is not yet known.

## Next step

Once a scan's timing is right-sized, move on to targeted `-sV`/NSE on the discovered ports rather than re-tuning further, per the two-stage methodology in [port-scanning-fundamentals](port-scanning-fundamentals.md).

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
