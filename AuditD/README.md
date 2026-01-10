# Auditd Web Server Detection Rules

This repository provides a **detection-oriented Auditd ruleset** designed for **Linux web servers**.  
The goal is to improve visibility and detection of **webshells, RCE, reverse shells, and post-exploitation activity** by monitoring critical system calls (`execve`, `connect`) for common attacker tools and interpreters.

These rules are **not compliance-focused**; they are optimized for **security detection and incident response**.

---

## ⚠️ IMPORTANT – Versioned Binaries (Critical)

Several interpreters and tools are **versioned on Debian and many other distributions**, for example:

- `/usr/bin/php8.4`
- `/usr/bin/ruby3.3`
- `/usr/bin/python3.11`
- `/usr/bin/socat1`

⚠️ **Auditd does NOT support wildcards** for the `exe=` filter.

This means you **must explicitly list and maintain** the correct binary paths that exist on your system.  
If a versioned binary is missing from the ruleset, **events will NOT be logged**.

### Recommended verification commands

```bash
readlink -f $(which php)
readlink -f $(which ruby)
readlink -f $(which python3)
readlink -f $(which socat)
```

❗ **Failure to update versioned paths WILL result in missed detections.**

---

## ⚠️ IMPORTANT – Web User UID (Critical)

The rules in this repository assume:

- **`uid=33`** → `www-data` (Debian / Ubuntu default)

This UID is used to detect:

- command execution from the web context
- outbound network connections from web applications

### Other distributions

On non-Debian systems, the web server user may be different, for example:

- `apache`
- `httpd`
- `nginx`
- `www`

You **MUST verify the actual UID** used by your web server and adjust the rules accordingly.

### Quick verification

```bash
getent passwd www-data apache nginx httpd 2>/dev/null
id -u www-data
id -u apache
id -u nginx
```

### PHP-FPM note

If you use **PHP-FPM**, especially with **custom pools**, the effective UID may be:

- the pool user
- NOT the main web service account

Make sure the rules match the **actual runtime UID**.

---

## Scope of Detection

This ruleset focuses on detecting:

- Webshell execution (`uid=33` execve)
- Reverse shells and outbound C2 connections
- Abuse of common tools:
  - shells (`sh`, `bash`, `dash`)
  - netcat / ncat / busybox
  - socat
  - curl / wget (download & execute)
  - Python, Perl, PHP, Ruby, Node.js
  - FIFO-based shells (`mkfifo`)
- Post-exploitation behavior (root and non-root)

---

## Design Philosophy

- **Low-level syscall visibility** (`execve`, `connect`)
- **No reliance on command history**
- **Resilient to argument reordering**
- **Effective even when PROCTITLE is hex-encoded**
- Optimized for **web servers**, not general-purpose desktops

---

## Deployment Notes

- Review and adapt paths before deployment
- Test with `auditctl -l` and `ausearch`
- Validate detection using `wazuh-logtest` (if using Wazuh)
- Consider removing unused language/tool blocks if not installed

---

## Disclaimer

These rules **increase visibility and security**, but they also increase logging volume.  
Always test in staging before deploying to production.

---

## Recommended Usage

- Internet-facing web servers
- API backends
- Application servers
- Containers / VMs running web workloads

---

## License

Use, modify, and adapt freely.  
No warranty — security detection always requires tuning per environment.
