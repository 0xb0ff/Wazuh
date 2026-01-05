# 🛡️ Wazuh Rule – SSH Brute Force Followed by Successful Login

## Rule ID
**100100**

## Severity
**Level 15 (High / Critical)**

---

## 📖 Overview

This Wazuh rule detects a **successful SSH login that is preceded by multiple authentication failures** from the **same source IP address** within a short time window.

This behavior is a strong indicator of a **successful brute-force attack**, where an attacker repeatedly attempts credentials until valid access is obtained.

---

## 🔍 Detection Logic

The rule triggers when **all of the following conditions** are met:

- **10 or more SSH authentication failures**
- Occurring within a **5-minute window (300 seconds)**
- Followed by a **successful SSH login**
- Targeting the **same user account**
- Originating from the **same source IP address**

This correlation significantly reduces false positives and focuses on **confirmed compromise scenarios**.

---

## ⚙️ Rule Configuration

```xml
<rule id="100000" level="15" frequency="10" timeframe="300">
  <if_sid>5715</if_sid>
  <if_matched_sid>5760</if_matched_sid>
  <same_srcip />
  <description>
    Multiple failed logins followed by a successful login for user $(dstuser) from $(srcip).
  </description>
  <mitre>
    <id>T1110</id>
    <id>T1021.004</id>
    <id>T1078.001</id>
    <id>T1110.001</id>
    <id>T1078</id>
  </mitre>
  <info type="text">
    Triggers when an IP fails SSH authentication 10+ times within 5 minutes, then successfully logs in to the same account.
  </info>
</rule>
```

---

## 🧪 Example Scenario

```text
Source IP: 203.0.113.45
User: admin

- 12 failed SSH login attempts
- Within 3 minutes
- Followed by a successful SSH login
```

➡️ **Alert is triggered** due to confirmed credential compromise behavior.

---

## ❌ Non-Triggering Scenarios

- Multiple failed logins without success
- Successful login without prior failures
- Failures spread over a long time period
- Failures from different source IPs

---

## 🎯 MITRE ATT&CK Mapping

| Technique ID | Name |
|-------------|------|
| **T1110** | Brute Force |
| **T1110.001** | Brute Force: Password Guessing |
| **T1021.004** | Remote Services: SSH |
| **T1078** | Valid Accounts |
| **T1078.001** | Valid Accounts: Local Accounts |

---

## 🚨 Alert Interpretation & Response

### Recommended Response Actions
1. Isolate the affected host
2. Disable or lock the compromised account
3. Reset credentials and rotate keys
4. Review SSH logs and active sessions
5. Investigate lateral movement or persistence

---

## 📌 Summary

> **This rule provides high-confidence detection of successful SSH brute-force attacks by correlating repeated authentication failures with a subsequent successful login from the same source IP.**
