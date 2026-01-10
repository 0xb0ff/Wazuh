# Wazuh Web Shell Detection Rules (auditd)

This repository contains **high-confidence Wazuh rules** designed to detect **web shell activity** on Linux web servers using **auditd** telemetry.

The rules focus on **runtime behavior**, not signatures, and are intended for **Apache / Nginx + PHP** environments exposed to the Internet.

---

## ✅ Prerequisites (MANDATORY)

⚠️ **auditd is a strict prerequisite**

These rules **WILL NOT WORK** without `auditd`.  
They rely on **kernel-level telemetry** (`execve`, `connect`) provided by auditd.

### Required components

- ✅ `auditd` installed and running on the host
- ✅ Wazuh **agent** installed on the host
- ✅ Wazuh **manager** receiving audit events

---

## 🔧 Wazuh Agent Configuration (REQUIRED)

The Wazuh agent **must** be configured to ingest audit logs.

On the **agent**, ensure the following configuration is present in  
`/var/ossec/etc/ossec.conf`:

```xml
<ossec_config>
  <localfile>
    <location>/var/log/audit/audit.log</location>
    <log_format>audit</log_format>
  </localfile>
</ossec_config>
```

After applying the configuration, restart the agent:

```bash
sudo systemctl restart wazuh-agent
```

---

### Verification

You should see audit events arriving on the manager:

```bash
grep audit /var/ossec/logs/archives/archives.log
```

If no events appear, verify:

- `auditd` is running
- permissions on `/var/log/audit/audit.log`
- the agent is connected to the manager

---

## 📌 Rules Overview

### Rule `100520` — Command execution (web shell)

```xml
<rule id="100520" level="12">
  <if_sid>80700</if_sid>
  <field name="audit.key">apache-exec</field>
  <description>[Command execution ($(audit.exe))]: Possible web shell attack detected</description>
  <mitre>
    <id>T1505.003</id>
    <id>T1059.004</id>
  </mitre>
</rule>
```

**What it detects**

- Command execution (`execve`) performed by a web server process
- Typical indicators of **RCE or web shell execution**

**Why it is high confidence**

- Web servers should not spawn shells or arbitrary commands
- Very low false-positive rate when audit rules are correctly scoped

---

### Rule `100521` — Network connection from web shell

```xml
<rule id="100521" level="12">
  <if_sid>80700</if_sid>
  <field name="audit.key">webshell-net-connect</field>
  <description>[Network connection via $(audit.exe)]: Possible web shell attack detected</description>
  <mitre>
    <id>TA0011</id>
    <id>T1049</id>
    <id>T1505.003</id>
  </mitre>
</rule>
```

**What it detects**

- Outbound network connections initiated by a web server process
- Common in **reverse shells, C2 callbacks, or payload staging**

**Why it is critical**

- Web servers rarely initiate outbound connections
- Strong indicator of **post-exploitation activity**

---

## ⚠️ IMPORTANT — UID WARNING (READ THIS)

These rules **depend on auditd rules that scope execution by UID**.  
⚠️ **Using the wrong UID will either miss attacks or create excessive noise**.

### Web server UID differs by distribution

| Distribution                 | Web Server User | Typical UID |
| ---------------------------- | --------------- | ----------- |
| Debian / Ubuntu              | www-data        | **33**      |
| RHEL / CentOS / Rocky / Alma | apache          | **48**      |
| Fedora                       | apache          | **48**      |
| openSUSE                     | wwwrun          | **30**      |
| Alpine Linux                 | apache          | **100**     |

❗ **Do NOT hardcode UID blindly**

Always verify on the target system:

```bash
ps -eo user,uid,comm | egrep 'apache|httpd|nginx'
id www-data
id apache
id nginx
```

---

## 🔧 Example auditd rules (Apache – Debian/Ubuntu)

```bash
# Command execution from web server
-a always,exit -F arch=b64 -S execve -F uid=33 -k apache-exec

# Outbound network connections from web server
-a always,exit -F arch=b64 -S connect -F uid=33 -k webshell-net-connect
```

➡️ Adjust `uid=33` according to your distribution.

---

## 🧠 Best Practices

- ✅ Keep auditd rules **broad**, filter in **Wazuh**
- ✅ Correlate command execution **AND** network activity
- ❌ Do NOT rely on file hashes or static payloads
- ❌ Do NOT silence shell executions from web servers

---

## 🧩 MITRE ATT&CK Mapping

- **T1505.003** — Server-Side Web Shell
- **T1059.004** — Command-Line Interface
- **TA0011** — Command and Control
- **T1049** — System Network Connections Discovery

---

## 🧪 Testing

Recommended testing approach:

- Use controlled `sudo -u <webuser>` command execution
- Validate with `ossec-logtest`
- Avoid deploying real web shells on production systems

---

## ⚠️ Disclaimer

These rules are intended for **defensive security monitoring** only.  
Test in a **lab environment** before deploying to production.

---

## 📄 License

MIT
