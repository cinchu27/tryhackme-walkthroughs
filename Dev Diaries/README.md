# 👨‍💻 Dev Diaries — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** OSINT / Git History Analysis
**Tags:** `osint` `github-recon` `git-log` `subdomain-enumeration` `crt.sh` `certificate-transparency` `commit-history` `git-patch`

---

## Overview

Dev Diaries simulates a realistic scenario: a client hired a freelance developer to build a website. The developer disappeared without handing over the source code. Your only starting point is the website's primary domain: **marvenly.com**.

Your mission is to investigate online development traces — subdomains, GitHub repositories, and Git commit history — to uncover hidden information and recover a flag. No exploitation, no shells — this is pure OSINT and Git forensics.

---

## Scenario

> *"We have just launched a website developed by a freelance developer. The source code was not shared with us, and the developer has since disappeared without handing it over. Despite this, traces of the development process and earlier versions of the website may still exist online. You are only given the website's primary domain as a starting point: marvenly.com"*

---

## Step 1 — Subdomain Enumeration

The main site may be down or unhelpful. Start by finding subdomains — especially development/staging environments that developers often leave exposed.

### Method 1 — Certificate Transparency Logs (crt.sh)

Certificate Transparency (CT) logs record every SSL/TLS certificate issued for a domain. Developers often spin up subdomains (like `dev.`, `staging.`, `uat.`) and register SSL certificates for them — leaving a permanent public trail.

```
https://crt.sh/?q=marvenly.com
```

Or search directly:
```
https://crt.sh/?q=%25.marvenly.com
```

The `%25` is URL-encoded `%`, which acts as a wildcard to find all subdomains.

**What to look for:** Any subdomain beyond `www.` — especially `dev.`, `staging.`, `uat.`, `test.`, `beta.`

### Method 2 — Subfinder (CLI Tool)

```bash
subfinder -d marvenly.com -o subdomains.txt
```

**Flags explained:**
- `-d` — Target domain
- `-o` — Output file

### Method 3 — Merklemap

```
https://www.merklemap.com/search?query=marvenly.com
```

Merklemap indexes certificate transparency logs and is fast for subdomain discovery.

### Method 4 — Amass

```bash
amass enum -d marvenly.com -o amass_results.txt
```

**Expected finding:** A development subdomain such as `dev.marvenly.com` — this is where the development version of the site is hosted.

---

## Step 2 — Investigating the Development Subdomain

Navigate to the discovered subdomain in your browser:

```
http://dev.marvenly.com
```

Inspect the page source carefully:

```bash
curl -s http://dev.marvenly.com | grep -i "github\|git\|source\|repo\|comment"
```

**What to look for:**
- HTML comments referencing the developer
- Links to GitHub repositories
- Developer username embedded in code comments

**Expected finding:** A comment or reference pointing to the developer's **GitHub username**.

---

## Step 3 — GitHub Reconnaissance

With the developer's GitHub username, search for their repositories:

```
https://github.com/<developer_username>
```

Or search GitHub directly:
```
https://github.com/search?q=marvenly&type=repositories
```

Look for a repository named something like **marvenly_site** or similar.

### Inspecting the Repository

Once you find the repository:

1. Check the **README** for any information
2. Look at the **repository description** and **topics**
3. Check if the source code is still present or was removed

---

## Step 4 — Git Commit History Analysis

Even if the source code was deleted from the repository, **Git never forgets**. Every commit is permanently stored and accessible.

### Viewing Commit History on GitHub

Navigate to the repository and click **"X commits"** at the top of the file list to see the full commit log.

Look at each commit message — they often reveal:
- Why code was removed
- The developer's reasoning
- Hidden data in diffs

### Cloning for Deeper Inspection

```bash
git clone https://github.com/<username>/marvenly_site
cd marvenly_site
```

**View full commit log:**
```bash
git log
```

**View commit log with changes:**
```bash
git log --oneline --all
git log -p
```

**Flags explained:**
- `--oneline` — Compact one-line format per commit
- `--all` — Show all branches including remote
- `-p` — Show patch/diff for each commit

**View a specific commit:**
```bash
git show <commit_hash>
```

**Check out a deleted file from history:**
```bash
git checkout <commit_hash> -- <filename>
```

**Expected finding:** A commit message explaining that the project was abandoned due to a **payment dispute** between the developer and client.

---

## Step 5 — Extracting the Developer's Email via .patch Trick

Git commit metadata includes the author's email, but GitHub hides it in the web UI. There's a classic trick to reveal it: **add `.patch` to any commit URL**.

### Normal commit URL:
```
https://github.com/<username>/marvenly_site/commit/<hash>
```

### Patch URL (reveals email):
```
https://github.com/<username>/marvenly_site/commit/<hash>.patch
```

**Output format:**
```
From: Developer Name <developer@email.com>
Date: Mon, 10 Jan 2023 14:22:11 +0000
Subject: [PATCH] Initial commit
```

The `From:` field reveals the developer's **full name and email address**.

---

## Step 6 — Recovering the Hidden Flag from Deleted Code

Even though the source code was removed in a later commit, the content still exists in Git history.

### Find the commit that removed files:
```bash
git log --diff-filter=D --summary
```

- `--diff-filter=D` — Show only commits that deleted files

### Recover deleted file content:
```bash
git show <commit_before_deletion>:<filename>
```

Or view the diff of the deletion commit:
```bash
git show <deletion_commit_hash>
```

The removed code contains a **hidden flag** embedded as a comment or variable in the source.

---

## Step 7 — Flag

The flag is hidden inside the deleted source code, recoverable through Git history:

```
THM{...}
```

---

## Tools Summary

| Tool | Purpose |
|------|---------|
| crt.sh | Certificate transparency subdomain discovery |
| subfinder | Automated subdomain enumeration |
| Merklemap | CT log search engine |
| Browser | Inspect dev subdomain page source |
| GitHub Search | Find developer's repository |
| git log | View commit history |
| git show | Inspect individual commits |
| git checkout | Recover deleted files from history |
| .patch URL trick | Extract developer email from commit metadata |

---

## OSINT + Git Methodology

```
Domain → CT logs (subdomains) → Dev subdomain → 
Page source (GitHub username) → Repository → 
Commit history → Deleted code recovery → Flag
```

---

## Key Takeaways

- **Certificate Transparency logs** (crt.sh) are one of the most powerful passive recon tools — every SSL cert ever issued for a domain is publicly logged
- **Development subdomains** (`dev.`, `staging.`, `uat.`) are frequently left exposed with sensitive information
- **Git never truly deletes data** — even removed files are recoverable from commit history using `git show` and `git checkout`
- The **`.patch` URL trick** on GitHub exposes commit author email addresses that are hidden in the web UI — a classic OSINT technique
- **Commit messages** are intelligence — developers write candid notes about why changes were made, disputes, and context that wouldn't appear anywhere else
- Sensitive data removed from a public repo in a later commit is still 100% publicly accessible through that repo's history — the only safe fix is to delete the entire repository

---

*Room link: https://tryhackme.com/room/devdiaries*
