# 🛡️ Snort — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Medium
**Category:** Blue Team / IDS / Network Security
**Tags:** `snort` `ids` `ips` `pcap` `network-analysis` `intrusion-detection` `snort-rules` `defensive-security`

---

## Overview

Unlike most TryHackMe rooms that focus on offensive techniques, Snort is a **blue team room** centred entirely on network intrusion detection and prevention. You'll learn to use Snort — the world's most widely deployed open-source IDS/IPS — across its three operating modes, analyze PCAP files for malicious traffic, and write custom detection rules from scratch. This room is essential for anyone pursuing a SOC Analyst or defensive security role.

---

## What Is Snort?

Snort is an open-source **Network Intrusion Detection System (NIDS)** and **Intrusion Prevention System (NIPS)** developed by Sourcefire (now Cisco). It inspects network packets in real time against a ruleset and can:

- **Detect** — Alert on suspicious traffic
- **Log** — Capture packets for forensic analysis
- **Prevent** — Drop malicious packets inline (IPS mode)

---

## Snort Operating Modes

### Mode 1 — Sniffer Mode

Simply captures and displays packets on screen. No analysis, no logging.

```bash
sudo snort -v
```

**Flags explained:**
- `-v` — Verbose: display packet headers on screen

```bash
sudo snort -vd
```

- `-d` — Display packet data (payload)

```bash
sudo snort -vde
```

- `-e` — Display link layer (Ethernet) headers

**Example — Sniff on a specific interface:**
```bash
sudo snort -v -i eth0
```

- `-i eth0` — Specify network interface

---

### Mode 2 — Packet Logger Mode

Captures packets and saves them to disk for later analysis.

```bash
sudo snort -dev -l /var/log/snort
```

**Flags explained:**
- `-l` — Log directory path
- `-dev` — Combined verbose, data, and link layer

**Log in binary (tcpdump-compatible) format:**
```bash
sudo snort -dev -l /var/log/snort -b
```

- `-b` — Log in binary pcap format (smaller, faster)

**Read logged packets back:**
```bash
sudo snort -r /var/log/snort/snort.log.<timestamp>
```

- `-r` — Read from pcap file

**Filter packets while logging:**
```bash
sudo snort -dev -l /var/log/snort -K ASCII
```

- `-K ASCII` — Log in ASCII format (human-readable, larger files)

---

### Mode 3 — IDS/IPS Mode (NIDS)

The core operational mode. Snort inspects traffic against rules and generates alerts.

```bash
sudo snort -c /etc/snort/snort.conf -l /var/log/snort
```

**Flags explained:**
- `-c` — Path to Snort configuration file
- `-l` — Log/alert output directory

**Run against a PCAP file (offline analysis):**
```bash
sudo snort -c /etc/snort/snort.conf -r capture.pcap -l /var/log/snort
```

**Run in alert-only mode (no logging):**
```bash
sudo snort -c /etc/snort/snort.conf -A console
```

- `-A console` — Print alerts to the terminal in real time

**Alert modes:**
```bash
-A fast       # Fast alert format (timestamp, message, IPs, ports)
-A full       # Full packet details in alert
-A console    # Print to stdout
-A none       # Disable alerting (log only)
```

**Testing your config:**
```bash
sudo snort -c /etc/snort/snort.conf -T
```

- `-T` — Test configuration and exit (no packet capture)

---

## Analyzing PCAP Files

Reading and investigating PCAP files is a key skill for SOC work.

### Basic PCAP Read

```bash
sudo snort -r capture.pcap
```

### Read with Full Rule Analysis

```bash
sudo snort -c /etc/snort/snort.conf -r capture.pcap -l /var/log/snort -X
```

- `-X` — Display raw packet data in hex and ASCII

### Count Packets in PCAP

```bash
sudo snort -r capture.pcap --daq-var buffer-timeout=0 2>&1 | grep "packets processed"
```

### Filter by Protocol During Read

```bash
sudo snort -r capture.pcap 'tcp'
sudo snort -r capture.pcap 'icmp'
sudo snort -r capture.pcap 'udp and port 53'
```

(BPF filter syntax applied at the end)

---

## Writing Snort Rules

Snort rules are the heart of the IDS. Every rule follows this structure:

```
[action] [protocol] [src_ip] [src_port] -> [dst_ip] [dst_port] (rule options)
```

### Rule Actions

| Action | Behaviour |
|--------|-----------|
| `alert` | Generate alert and log packet |
| `log` | Log packet only |
| `drop` | Block and log packet (IPS mode) |
| `reject` | Block, log, and send TCP reset |
| `pass` | Ignore packet |

---

### Example Rules

**Alert on any ICMP traffic (ping detection):**
```
alert icmp any any -> any any (msg:"ICMP Packet Detected"; sid:1000001; rev:1;)
```

**Alert on HTTP traffic to a specific IP:**
```
alert tcp any any -> 192.168.1.1 80 (msg:"HTTP Request to Target"; sid:1000002; rev:1;)
```

**Detect SSH login attempts:**
```
alert tcp any any -> any 22 (msg:"SSH Connection Attempt"; flags:S; sid:1000003; rev:1;)
```

- `flags:S` — Match TCP SYN packets only (connection attempts)

**Detect a specific string in HTTP traffic:**
```
alert tcp any any -> any 80 (msg:"Possible SQL Injection Attempt"; content:"UNION SELECT"; nocase; sid:1000004; rev:1;)
```

- `content:` — Match specific string in payload
- `nocase` — Case-insensitive matching

**Detect large ICMP packets (possible tunnel or DoS):**
```
alert icmp any any -> any any (msg:"Large ICMP Packet"; dsize:>512; sid:1000005; rev:1;)
```

- `dsize:>512` — Match packets with data size greater than 512 bytes

**Detect FTP login attempts:**
```
alert tcp any any -> any 21 (msg:"FTP Login Attempt"; content:"USER"; sid:1000006; rev:1;)
```

---

### Rule Options Reference

| Option | Description | Example |
|--------|-------------|---------|
| `msg:` | Alert message | `msg:"Alert Name";` |
| `sid:` | Unique rule ID (>1000000 for custom) | `sid:1000001;` |
| `rev:` | Rule revision number | `rev:1;` |
| `content:` | Match string in payload | `content:"GET";` |
| `nocase` | Case-insensitive content match | `nocase;` |
| `flags:` | TCP flag match | `flags:SA;` |
| `dsize:` | Packet data size | `dsize:>100;` |
| `ttl:` | IP TTL value | `ttl:<10;` |
| `threshold:` | Limit alert frequency | `threshold: type limit, track by_src, count 5, seconds 60;` |

---

### Where to Add Custom Rules

```bash
sudo nano /etc/snort/rules/local.rules
```

Add your custom rules here. Reference this file in `snort.conf`:

```bash
include $RULE_PATH/local.rules
```

---

## Reading Alert Files

After running Snort in IDS mode, check the alert output:

```bash
cat /var/log/snort/alert
```

Or tail it in real time:

```bash
tail -f /var/log/snort/alert
```

**Alert format (fast mode):**
```
[**] [1:1000001:1] ICMP Packet Detected [**]
[Priority: 0]
06/23-14:22:13.123456 192.168.1.5 -> 192.168.1.1
ICMP TTL:64 TOS:0x0 ID:1234 IpLen:20 DgmLen:84
```

---

## Summary

| Task | Command |
|------|---------|
| Sniffer mode | `snort -v -i eth0` |
| Packet logger | `snort -dev -l /var/log/snort` |
| IDS mode (live) | `snort -c snort.conf -l /var/log/snort -A console` |
| Analyze PCAP | `snort -c snort.conf -r file.pcap -l /var/log/snort` |
| Test config | `snort -c snort.conf -T` |
| Edit custom rules | `nano /etc/snort/rules/local.rules` |

---

## Key Takeaways

- Snort's three modes serve different purposes — sniffer for quick inspection, logger for forensics, IDS for active monitoring
- Writing effective rules requires understanding both the protocol structure and the attack pattern you're targeting
- `sid` values above `1000000` are reserved for local/custom rules — never overlap with official Snort rule SIDs
- The `-A console` flag is extremely useful for real-time alert feedback during testing
- PCAP analysis with Snort is a core SOC skill — you'll use this methodology in real incident response scenarios
- Defensive security knowledge makes you a better attacker, and vice versa — understanding how IDS rules work helps you evade them during red team engagements

---

*Room link: https://tryhackme.com/room/snort*
