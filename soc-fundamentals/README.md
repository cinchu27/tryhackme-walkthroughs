# 🛡️ TryHackMe Walkthrough – SOC Fundamentals

## 📌 Room Information

- **Room Name:** SOC Fundamentals  
- **Platform:** TryHackMe  
- **Difficulty:** Easy  
- **Category:** Blue Team / SOC  
- **Focus:** SOC concepts, people, process, and technology  

---

## 🎯 Room Objective

The **SOC Fundamentals** room provides a foundation in how a **Security Operations Center (SOC)** works.  
A SOC is a dedicated team that continuously **monitors an organization’s network and systems**, detects suspicious activity, and responds to security events — often 24/7. 

### Learning Objectives Include:

- Building a **baseline understanding of SOC**
- Exploring **detection and response capabilities**
- Learning about the three pillars — **People, Processes, Technology**
- Completing a **practical SOC investigation exercise** 

---

## 🧩 Task 1 – Introduction to SOC

This task introduces the concept of a SOC and outlines the general objectives of this room.

### ❓ Question  
**What does the term SOC stand for?**

### ✅ Answer  
Security Operations Center

---

## 🧩 Task 2 – Purpose and Components

This task explains the core purpose of a SOC, with a focus on **detection and response**, and outlines the three foundational pillars of a SOC. 

### ❓ Question 1  
**The SOC team discovers an unauthorized user is trying to log in to an account. Which capability of SOC is this?**

### ✅ Answer  
Detection

### ❓ Question 2  
**What are the three pillars of a SOC?**

### ✅ Answer  
People, Process, Technology

---

## 🧩 Task 3 – People

This task focuses on the SOC team structure and the roles within a SOC that help detect and respond to threats. 

### ❓ Question 1  
**Alert triage and reporting is the responsibility of?**

### ✅ Answer  
SOC Analyst (Level 1)

### ❓ Question 2  
**Which role in the SOC team allows you to work dedicatedly on establishing rules for alerting security solutions?**

### ✅ Answer  
Detection Engineer

---

## 🧩 Task 4 – Process

This task covers SOC processes such as **alert triage, reporting, incident response, and forensics**, and introduces the **5 Ws** approach analysts use during investigations.

### ❓ Question 1  
**At the end of the investigation, the SOC team found that John had attempted to steal the system's data. Which “W” from the 5 Ws does this answer?**

### ✅ Answer  
Who

### ❓ Question 2  
**The SOC team detected a large amount of data exfiltration. Which “W” from the 5 Ws does this answer?**

### ✅ Answer  
What

---

## 🧩 Task 5 – Technology

This task provides an overview of key security technologies used in a SOC, including SIEM, EDR, and Firewalls. 

### ❓ Question 1  
**Which security solution monitors the incoming and outgoing traffic of the network?**

### ✅ Answer  
Firewall

### ❓ Question 2  
**Do SIEM solutions primarily focus on detecting and alerting about security incidents? (yea/nay)**

### ✅ Answer  
yea

---

## 🧩 Task 6 – Practical Exercise of SOC

In this hands-on task, you take the role of a **Level 1 SOC Analyst** investigating an alert in a simulated SIEM dashboard. The goal is to answer the **5 Ws** for a port scan alert. 

### 🧪 Scenario Summary

- You receive an alert: **Port scanning activity** was observed on the network.  
- The **vulnerability assessment team** confirms they ran a scan from host `10.0.0.8`.  
- You use the SIEM to review logs and answer the investigation questions. :contentReference[oaicite:8]{index=8}

---

### ❓ Investigation Questions & Answers

**What: Activity that triggered the alert?**  
Port Scan

**When: Time of the activity?**  
June 12, 2024 17:24

**Where: Destination host IP?**  
10.0.0.3

**Who: Source host name?**  
Nessus

**Why: Reason for the activity? Intended/Malicious**  
Intended

**Additional Investigation Notes: Has any response been sent back to the port scanner IP? (yea/nay)**

yea

**What is the flag found after closing the alert?**

THM{000_INTRO_TO_SOC}


---

## 🧩 Task 7 – Conclusion

This task concludes the SOC Fundamentals room and summarizes the key concepts learned.

---

## 🏁 Final Thoughts

The **SOC Fundamentals** room provides a strong foundation in:

- SOC structure and purpose  
- Blue team roles and workflows  
- Alert triage and investigation  
- SOC technologies and processes  

This room is an excellent starting point for anyone interested in:

- SOC Analyst roles  
- Defensive Security  
- Incident Response  
- Blue Team operations  

---

## 📂 Repository Note

This walkthrough is part of my **TryHackMe Walkthroughs** GitHub repository, created for:

- Learning reinforcement  
- Documentation  
- Portfolio demonstration  

> ⚠️ This write-up is intended for educational purposes only.

