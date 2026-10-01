# Web Shells and Default Webroots

**Phase:** 04-foothold-and-shells
**Platform:** Web
**Source module(s):** 02 Getting Started

## When to use

When a web application allows an upload or write primitive (image upload, plugin install, file manager) that can be abused to drop server-side code, as happened on the Nibbles box.

## Prerequisites

A write primitive into a path the web server will execute (usually one matching the server's script-handler config, e.g. `.php`).

## Command

```php
<?php system($_REQUEST["cmd"]); ?>
```

```php
<?php system('id'); ?>
```

Default webroots to check for where an upload landed:

| Server | Path |
|---|---|
| Apache | `/var/www/html/` |
| Nginx | `/usr/local/nginx/html/` |
| IIS | `c:\inetpub\wwwroot\` |

## Expected output

Visiting the uploaded file's URL (optionally with `?cmd=id` for the request-based version) runs the command and returns its output in the page response.

## What failure looks like

An upload that throws a visible error or warning does not necessarily mean the file failed to save or execute. On the Nibbles box, the upload feature threw a broken-image-processing warning while still saving and executing the payload. Check the actual upload path and try hitting the file directly rather than trusting the error message.

## Why it works

Any application that accepts a file and later serves it from a path the web server executes as code (rather than serves as a static file) will run whatever is in that file. An "image upload" feature that does not validate file content (only an extension, or nothing at all) is a direct path to code execution if a `.php` (or equivalent) file can be smuggled through it.

## Alternatives and tradeoffs

- `$_REQUEST["cmd"]` version: interactive, repeatable, lets different commands be run without re-uploading.
- `system('id')` version: simpler one-shot proof of execution, used first to confirm the vector works before uploading the fuller interactive version.
- A web shell is inherently clunky for anything beyond simple commands (no persistent working directory feel, no easy interactivity); the first real priority after confirming code execution is usually to pivot to a proper reverse shell, see [shell-types-and-upgrade](shell-types-and-upgrade.md).

## Next step

Use the web shell to trigger a reverse shell listener rather than operating through it long-term. Uploaded or dropped files often land in paths already found during earlier directory enumeration, so check those first rather than guessing fresh paths.

## Self-test

- [ ] I can explain it without notes
- [ ] I can predict the output before running it
- [ ] I can recognize a wrong result
- [ ] I can troubleshoot a failure
- [ ] I can reproduce it from the documentation if I forget the syntax
