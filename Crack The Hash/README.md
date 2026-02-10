# 🔐 Crack the Hash — TryHackMe Writeup

This room focuses on identifying and cracking different types of hashes using both online tools and local cracking utilities such as **Hashcat**.  
It is a beginner-friendly room designed to introduce common hash formats and cracking techniques.

---

## 🧠 Room Information

| Field | Value |
|------|------|
| Room Name | Crack the Hash |
| Platform | TryHackMe |
| Difficulty | Easy |
| Focus | Hash Identification & Cracking |
| Wordlist | rockyou.txt |

---

## 🛠 Tools Used

- **Hashcat**
- **rockyou.txt**
- Online hash cracking tools (for unsupported cases)

---

## 🧪 Task 1: Level 1

Crack the following hashes using hints provided.

---

### Hash 1

48bb6e862e54f2a795ffc4e541caed4d

- Type: MD5

```bash
hashcat -m 0 hash1_1.txt /usr/share/wordlists/rockyou.txt --show
```
Answer: easy

### Hash 2

CBFDAC6008F9CAB4083784CBD1874F76618D2A97


- Type: SHA1

```bash
hashcat -m 100 hash1_2.txt /usr/share/wordlists/rockyou.txt --show
```
Answer: password123

### Hash 3

1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032

- Type: SHA256

```bash
hashcat -m 1400 hash1_3.txt /usr/share/wordlists/rockyou.txt --show
```
Answer: letmein

### Hash 4

$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom

- Type: bcrypt

```bash
hashcat -m 3200 hash1_4.txt /usr/share/wordlists/rockyou.txt --force
```
Answer: bleh

### Hash 5

279412f945939ba78ce0758d3fd83daa

- Type: MD4

```bash
hashcat -m 900 hash1_5.txt /usr/share/wordlists/rockyou.txt
```
This hash was not cracked using rockyou.txt, so an online tool was used.

Answer: Eternity22

----

## 🧪 Task 2: Level 2

----

This level increases the difficulty.

All passwords are still present in the rockyou wordlist.

### Hash 1

F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85

- Type: SHA256

```bash
hashcat -m 1400 hash2_1.txt /usr/share/wordlists/rockyou.txt
```
Answer: paule

### Hash 2

1DFECA0C002AE40B8619ECF94819CC1B

- Rounds: 5
- Type: NTLM

```bash
hashcat -m 1000 hash2_2.txt /usr/share/wordlists/rockyou.txt
```
Answer: n63umy8lkf4i

### Hash 3

$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.

- Salt: aReallyHardSalt
- Type: SHA-512 crypt

```bash
hashcat -m 1800 hash2_3.txt /usr/share/wordlists/rockyou.txt --force
```
Answer: waka99

### Hash 4

e5d8870e5bdd26602cab8dbe07a942c8669e56d6


- Salt: tryhackme
- Type: SHA-512 unix

```bash
hashcat -m 1800 hash2_4.txt /usr/share/wordlists/rockyou.txt
```
Answer: 481616481616

----
📌 Summary

Crack the Hash is an excellent introductory room for learning:

- Hash identification

- Hashcat modes

- Using wordlists effectively

- When to rely on online tools vs local cracking

By completing this room, you gain hands-on experience with real-world password cracking techniques and a solid foundation for more advanced challenges.
