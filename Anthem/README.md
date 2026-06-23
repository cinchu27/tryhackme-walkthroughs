# ✉️ Anthem — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** CTF / Windows / OSINT
**Tags:** `nmap` `windows` `rdp` `cms` `umbraco` `osint` `credential-harvesting` `privilege-escalation`

---

## Overview

Anthem is one of TryHackMe's few beginner-friendly **Windows rooms**, making it a valuable contrast to the Linux-heavy CTF landscape. The room centres around a CMS (Umbraco) running on a Windows Server, with a creative OSINT challenge to discover the admin's username from a poem clue. The privilege escalation involves finding credentials hidden in a protected Windows file. A great introduction to Windows enumeration methodology.

---

## Reconnaissance

### Nmap — Port Scanning

```bash
nmap -sV -sC -oN nmap_anthem.txt <TARGET_IP>
```

**Flags explained:**
- `-sV` — Version detection
- `-sC` — Default script scan
- `-oN` — Save output

**Results:**
```
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0
3389/tcp open  ms-wbt-server Microsoft Terminal Services (RDP)
```

HTTP and RDP. A classic Windows web server setup.

Also run a UDP scan for good measure:
```bash
nmap -sU --top-ports 20 <TARGET_IP>
```

---

## Web Enumeration

### robots.txt

Always check `robots.txt` first:

```bash
curl http://<TARGET_IP>/robots.txt
```

**Output (key entries):**
```
Disallow: /bin/
Disallow: /config/
Disallow: /umbraco/
Disallow: /umbraco_client/
UmbracoIsTheBest!
```

The robots.txt disallows several CMS-related directories, confirming the site runs **Umbraco CMS**. The string `UmbracoIsTheBest!` looks like a password — save it.

### Identifying the CMS

Navigate to `http://<TARGET_IP>` — you'll see a basic blog-style site. Check:

- Page source for comments or metadata
- `/umbraco` for the admin login panel

```
http://<TARGET_IP>/umbraco
```

Confirms Umbraco CMS admin login.

---

## OSINT — Discovering the Admin Username

The room hides the admin's name in a **poem** published on the blog. Read the posts carefully. One article contains a poem — search for the poem's first line or distinctive phrase online:

```
"A cheery soul" + site:en.wikipedia.org
```

The poem is *"Solomon Grundy"* — and the author's name on the post matches a character from DC Comics. The admin's email format follows a pattern visible on the site (e.g., `Jane@anthem.com`).

Look at other author profiles on the site to understand the email naming convention, then apply it to the discovered name.

**Admin email discovered:** `SG@anthem.com` (or similar based on your OSINT)

---

## Flag Hunting — Web Flags

Several flags are hidden across the website. Enumerate thoroughly:

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x aspx,html,txt -t 40 -o gobuster_anthem.txt
```

**Flag locations to check:**
- Page source of the homepage
- `robots.txt` entries
- Individual blog post pages and their sources
- Author profile pages
- Meta tags in page headers:

```bash
curl -s http://<TARGET_IP>/ | grep -i "THM{"
curl -s http://<TARGET_IP>/archive/a-cheery-soul/ | grep -i "THM{"
```

Collect all web flags before moving to RDP. ✅

---

## RDP Access

Using the credentials discovered (email as username, `UmbracoIsTheBest!` as password):

### On Linux — Using xfreerdp

```bash
xfreerdp /u:SG /p:UmbracoIsTheBest! /v:<TARGET_IP>
```

**Flags explained:**
- `/u:` — Username
- `/p:` — Password
- `/v:` — Target IP

### On Linux — Using rdesktop

```bash
rdesktop -u SG -p UmbracoIsTheBest! <TARGET_IP>
```

You'll get a Windows desktop GUI. ✅

---

## Windows Enumeration

### Finding the User Flag

Once on the desktop, check common locations:

- Desktop
- Documents
- Downloads

```
C:\Users\SG\Desktop\user.txt
```

User flag captured. ✅

### Showing Hidden Files

Go to **File Explorer → View → Options → Change folder and search options → View tab → Show hidden files, folders, and drives**.

This reveals hidden files and folders that are invisible by default on Windows.

---

## Privilege Escalation — Credential Harvesting

### Finding the Hidden Backup File

With hidden files now visible, navigate to the `C:\` drive root and look for hidden folders:

```
C:\backup\
```

Inside you'll find a file — likely named `restore` or similar — that is **access-denied** when you try to open it.

### Taking Ownership of the File

Right-click the file → **Properties → Security → Advanced → Owner → Change**

Change the owner to `SG` (your current user). Then grant yourself **Full Control** under permissions.

Now open the file — it contains the **Administrator password**.

### Accessing the Administrator Account

With the Administrator password, either:

**Switch user via RDP:**
```
Username: Administrator
Password: <discovered_password>
```

Or use the **Run As** feature or open a new RDP session.

### Root Flag

Navigate to:
```
C:\Users\Administrator\Desktop\root.txt
```

Root flag captured. ✅

---

## Summary

| Step | Technique | Tool / Method |
|------|-----------|---------------|
| Reconnaissance | Port scan | Nmap |
| CMS identification | robots.txt analysis | curl / browser |
| Username discovery | OSINT on blog poem | Google / Wikipedia |
| Credential discovery | robots.txt password | Manual |
| Web flags | Source inspection + Gobuster | curl / browser |
| Remote access | RDP login | xfreerdp / rdesktop |
| Privilege escalation | File ownership takeover | Windows GUI |
| Admin credentials | Hidden backup file | Windows Explorer |

---

## Key Takeaways

- **robots.txt is goldmine** — it revealed both the CMS type and the password in this room
- Windows CTFs require a completely different mindset — think RDP, hidden files, file permissions, and registry
- OSINT is a legitimate and powerful skill — the admin's name was hidden in public content, not in any technical vulnerability
- Windows file ownership can be changed by local admins — always check ACLs on restricted files
- Umbraco is a common .NET CMS — knowing how to enumerate and identify CMS platforms speeds up your attack chain significantly

---

*Room link: https://tryhackme.com/room/anthem*
