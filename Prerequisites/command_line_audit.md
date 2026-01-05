# 🖥️ Linux Command Line Auditing via Bash and Rsyslog

This guide explains how to enable **command-line auditing** on a Linux system by logging every executed shell command to syslog and storing it in a dedicated log file.

⚠️ **Important**: This configuration applies to **interactive shells** and should be deployed with user awareness and compliance considerations.

---

## 🔐 Prerequisites

- Linux system with `bash`
- Root or sudo privileges
- `rsyslog` installed and enabled

---

## 1️⃣ Become Root

Log in to the Linux system and switch to the root account:

```bash
sudo su -
```

---

## 2️⃣ Configure Bash Command Logging

Edit `/etc/profile`:

```bash
vi /etc/profile
```

Add the following lines **at the bottom of the file**:

```bash
# --------------------------------------------------
# Command line audit logging (bash -> syslog)
# --------------------------------------------------

function log2syslog
{
   declare COMMAND
   COMMAND=$(fc -ln -0 | tr -d '\t')
   logger -p local1.notice -t bash_history -i -- "USER=${USER} PWD=${PWD} CMD=${COMMAND}"
}

trap log2syslog DEBUG
```

### 🔍 What This Does
- Captures **every executed command**
- Removes tab characters to avoid log artifacts
- Sends logs to **syslog facility `local1.notice`**
- Includes user, working directory, and command content

Save and exit `/etc/profile`.

---

## 3️⃣ Configure Rsyslog Output

Edit the rsyslog configuration file:

```bash
vi /etc/rsyslog.conf
```

Add the following lines **at the bottom of the file**:

```text
# --------------------------------------------------
# Command line audit logging
# --------------------------------------------------
local1.*    -/var/log/cmdline
```

### 📄 Result
All shell command logs sent to `local1` will be written to:

```
/var/log/cmdline
```

Save and exit `/etc/rsyslog.conf`.

---

## 4️⃣ Restart Rsyslog

Apply the changes by restarting the rsyslog service:

```bash
/etc/init.d/rsyslog restart
```

> 🔁 Alternatively, reboot the system to ensure all user sessions reload `/etc/profile`.

---

## 🧪 Example Log Entry

```text
bash_history: USER=root PWD=/root CMD=ls -lah
```

---

## 🔐 Security & Compliance Notes

- Commands containing **passwords or secrets** will be logged in clear text
- Users should be informed of command auditing
- This method can be bypassed by advanced users (`unset trap`, non-bash shells)
- For regulated environments, consider `auditd` or `tlog`

---

## ✅ Use Cases

- SOC monitoring
- Incident response
- Forensics
- Bastion hosts
- Honeypots
- Linux command accountability

---

## 📌 Summary

> This setup provides lightweight, real-time command auditing by leveraging native Bash hooks and rsyslog, offering high visibility with minimal overhead.

---
