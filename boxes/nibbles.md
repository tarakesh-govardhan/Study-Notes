# Nibbles (Retired, Easy, Linux)

**Status:** Retired. Both flags captured on first attempt.
**Vector:** Web (Nibbleblog file upload)
**PrivEsc:** Writable sudo-runnable script
**Feeds:** [04-foothold-and-shells](../methodology/04-foothold-and-shells/README.md), [05-privesc-linux](../methodology/05-privesc-linux/README.md)

## Attack chain summary

```
Nmap (22, 80 open)
  -> whatweb + curl -> HTML comment reveals the blog's install directory
  -> README file discloses the exact application version -> maps to a known Metasploit module
  -> Gobuster -> finds /admin.php, directory listing enabled on two other paths
  -> an XML config file confirms a valid username, no password
  -> a second XML file leaks site metadata -> the site name itself turns out to be the login password
  -> Admin panel -> Plugins -> image upload feature -> PHP code execution via upload
  -> reverse shell as a low-privileged user
  -> an archive in the home directory contains a script, writable and root-runnable via sudo NOPASSWD
  -> append a reverse shell one-liner to that script -> sudo execution -> root
```

## What I learned beyond the base module notes

**Check page source and HTML comments for hidden clues.** Developers leave directory hints, debug notes, and sometimes credentials directly in comments.

**README or CHANGELOG files often disclose exact version numbers.** After finding a CMS or app, check for a README. Confirming the exact version maps directly to known CVEs or exploit modules.

```bash
curl -s <url> | xmllint --format -        # pretty-print XML output
```

**Directory listing enabled is free enumeration.** If a redirecting directory has listing turned on, browse its contents directly instead of guessing filenames.

**Metadata-informed password guessing.** When no password is found directly, check site metadata: site name, box name, notification emails, theme names. Passwords are sometimes trivially tied to branding, so do not rely only on wordlist guessing.

**Check for lockout behavior before brute-forcing.** Verify there is no failed-login tracking or IP blacklisting before throwing a tool like Hydra at a login form. Otherwise it wastes time and can trigger a lockout.

**CMS and admin panels with an upload feature are worth testing for code execution.** Any image, file, or avatar upload is worth testing with a payload instead of a real file:
```php
<?php system('id'); ?>
```

**An error on upload does not mean the upload failed.** The feature threw a visible warning (broken image processing) while still saving and executing the file. Check the actual upload directory or response rather than trusting the error message alone.

**Uploaded or dropped files often land in paths already found during directory enumeration.** Earlier Gobuster results become the map for where to check post-exploitation.

**Safe editing practice when abusing a writable sudo-runnable script:**
```bash
echo '<payload>' | tee -a <script>        # append, don't overwrite
```
Always append, backing up first if possible, rather than overwriting, to avoid breaking the file or causing disruption on a real engagement.

**`unzip`**, first use this session, for basic archive extraction:
```bash
unzip <file>.zip
```

## Weak point found here (logged after first attempt)

**Never having used GTFOBins before** was the actual gap, not PHP knowledge. The file-upload payload was memorized correctly and the enumeration path to the Plugins feature was normal multi-vector work, not a miss. The gap showed up at escalation: had sudo rights on an interpreter but did not know to check GTFOBins for the escalation pattern, and needed AI assistance to get there.

Full detail and action items: [weak-points](../weak-points/README.md).

```bash
sudo php -r 'system("/bin/sh -i");'
```

What this does:
- `sudo /usr/bin/php`: only works because `sudo -l` showed PHP runnable as root with NOPASSWD. GTFOBins' pattern: if an interpreter (php, python, perl, etc.) is runnable via sudo, check GTFOBins for how that specific binary escalates to a shell.
- `-r '<code>'`: PHP flag to run inline code without writing a `.php` file.
- `system("...")`: the same function used in the web-shell payload earlier in the box; executes a shell command.

GTFOBins' version spawns a shell in-place, no network hop needed, since a session already exists on the box. The longer mkfifo/netcat reverse-shell chain used on the first attempt works too, but solves a problem that does not exist here (a new network connection) when a simpler in-session escalation was available.

**Lesson: do not default to the familiar tool (reverse shell one-liner) when a simpler GTFOBins-documented path exists. Check GTFOBins first whenever `sudo -l` shows a runnable interpreter or binary.**
