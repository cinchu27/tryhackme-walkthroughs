# 🧠 Psycho Break – TryHackMe Walkthrough

Help Sebastian and his team of investigators to withstand the dangers that come ahead.

VM Difficulty: Easy

Tags: Web, Enumeration, Steganography, Cryptography, Bruteforce, Reverse Engineering, Privilege Escalation

🧪 Room Info

TryHackMe Room: Psycho Break

Room Link: https://tryhackme.com/room/psychobreak

Machine OS: Ubuntu Linux

🛠️ Task 1 — Recon

How many ports are open?

Answer: 3

What is the operating system that runs on the target machine?

Answer: Ubuntu

🌐 Task 2 — Web

Key to the looker room

Answer: 532219a04ab7a02b56faafbec1a4c1ea

Key to access the map

Decode using Atbash cipher

Answer: Grant_me_access_to_the_map_please

The Keeper Key

Found in SafeHeaven/keeper via reverse image search

Answer: 48ee41458eb0b43bf82b986cecf3af01

What is the filename of the text file (without extension)?

Found in abandonedRoom via shell parameter abuse

Answer: you_made_it

🗃️ Task 3 — Help Mee

Who is locked up in the cell?

Answer: joseph

There is something weird with the .wav file. What does it say?

Answer: SHOWME

What is the FTP Username?

Answer: joseph

What is the FTP User Password?

Answer: intotheterror445

💻 Task 4 — Crack it open

The key used by the program

Bruteforcing with random.dic

Answer: kidman

What do the crazy long numbers mean when they’re decrypted?

They are Multi-tap (phone keypad) numbers

Answer: KIDMANSPASSWORDISSOSTRANGE

🏁 Task 5 — Go Capture The Flag

user.txt

Answer: 4C72A4EF8E6FED69C72B4D58431C4254

root.txt

Answer: BA33BDF5B8A3BFC431322F7D13F3361E

[Bonus] Defeat Ruvik

Answer: Delete the user ruvik (e.g. sudo deluser ruvik)

# 📌 Summary of Important Commands
# Nmap enumeration
sudo nmap -sV -sC -O -T4 TARGET_IP -oA psycho_break

# Gobuster for hidden dirs
gobuster dir -u http://TARGET_IP/SafeHeaven/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# Steghide to extract hidden data
steghide extract -sf Joseph_Oda.jpg -p SHOWME

📌 Notes

✔ Web enumeration involved checking HTML source comments and hidden paths.

✔ Directory scanning and reverse image search was key to getting the Keeper Key.

✔ Privilege escalation was possible via a world-writable cron script.

