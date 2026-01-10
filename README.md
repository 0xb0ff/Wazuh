## Auditd Web Server Detection Rules

Detection-focused **Auditd ruleset for Linux web servers**, designed to identify **webshells, RCE, reverse shells, and post-exploitation activity** using low-level system call monitoring.

This project provides **practical, attacker-aware audit rules** that focus on **how intrusions really happen on web servers**, not on compliance checklists.

---

## 🎯 Project Goal

The goal of this project is to **detect malicious behavior early**, especially:

- Remote Code Execution (RCE)
- Webshell activity
- Reverse shells and outbound C2 connections
- Abuse of scripting languages and common attacker tools
- Post-exploitation activity (root and non-root)

All detections are based on **kernel-level telemetry** (`execve`, `connect`) using **auditd**, making them resilient to:

- argument reordering
- shell obfuscation
- hex-encoded process titles
- missing command history

---

## 🧠 Detection Philosophy

This ruleset is built around a few core principles:

- **Behavior over signatures**
- **Syscalls over logs**
- **Execution + network = signal**
- **Web context matters**
- **Root activity is not ignored**

The rules are designed to answer questions like:

- *“Why is a web process executing a shell?”*
- *“Why is this interpreter opening an outbound TCP connection?”*
- *“Why is this tool spawning a shell at all?”*

---

## 🛡️ What This Detects

This project focuses on detecting abuse of commonly leveraged tools and techniques, including:

- Interactive shells (`sh`, `bash`, `dash`)
- Netcat / ncat / busybox
- socat
- curl / wget (download & execute)
- PHP (CLI and web context)
- Python, Perl, Ruby, Node.js
- FIFO-based reverse shells (`mkfifo`)
- Web user (`www-data`) command execution
- Root-level post-exploitation behavior

---

## 🧩 Designed For

- Internet-facing web servers
- Application servers
- API backends
- VM-based or bare-metal Linux hosts
- Security-focused environments (SOC / DFIR / blue team)

This is **not** intended for desktops or developer workstations.

---

## ⚠️ Important Notes

- Auditd does **not** support wildcards for binary paths (`exe=`).
- Versioned binaries **must be maintained per system**.
- Web server UID (`www-data`, `apache`, etc.) **must be verified**.
- These rules increase visibility and logging volume — test before production deployment.

Detailed deployment notes and caveats are documented in the project documentation.

---

## 📦 Project Structure

- **Auditd rules** grouped by attack surface (web, shells, interpreters, tools)
- **Wazuh rules** for alerting and escalation (optional)
- **Documentation** explaining required adaptations per distribution

---

## 🔍 Why This Exists

Most Auditd examples focus on:

- compliance
- change tracking
- generic logging

This project exists because **real-world web compromises require detection rules that understand attacker tradecraft**.

If an attacker needs to:

- spawn a shell
- open a socket
- download and execute code

…this ruleset is designed to notice.

---

## 📜 License

Free to use, adapt, and modify.  
No warranty — detection always requires tuning per environment.

---

## 🚀 Getting Started

Start by reading the deployment documentation and verifying:

- web server UID
- interpreter paths
- installed tools

Then load the rules, test with known payloads, and tune alert levels to your environment.
