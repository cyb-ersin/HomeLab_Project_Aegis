# Ch.03 — Network Forensics & PCAP Analysis

**homelab_AEGIS** · SOC from Scratch · April 11, 2026

---

## Overview

In Ch.01 we **detected** attacks. In Ch.02 we **blocked** them. Ch.03 asks a different question:

> *If you only had the raw network traffic — no SIEM, no logs — could you still reconstruct what happened?*

The answer: **yes.**

We replayed the Ch.01/02 attack scenario, captured all traffic with `tcpdump`, and analyzed it offline with Wireshark. The result: attacker IP, tools used, attack patterns, and even the successful login — all reconstructed from a single PCAP file.

---

## Lab Environment

| Machine | Role | OS | IP |
|:--------|:-----|:---|:---|
| MacBook Pro | Analysis (Wireshark) | macOS | 192.168.178.122 |
| Kali Laptop | Attacker | Kali Linux | 192.168.178.129 |
| aegis-sentinel VM | Capture Node | Ubuntu 24.04.4 LTS | 192.168.178.126 |

**Network:** Wireless Access Point · 192.168.178.0/24

---

## What Happened

```
aegis-sentinel: tcpdump starts → captures everything
        ↓
Kali: nmap -sS -A -T4     → SYN Scan
Kali: nmap -O             → OS Fingerprinting
Kali: nmap -sV            → Service Detection
Kali: hydra SSH           → Brute Force → password: 1111 ✅
        ↓
aegis-sentinel: tcpdump stops → 8936 packets saved
        ↓
MacBook: Wireshark → PCAP analysis
```

---

## What We Proved from the PCAP

| Question | Answer | Evidence |
|:---------|:-------|:---------|
| Who attacked? | 192.168.178.129 | 89.2% of all packets |
| Which tool? | nmap | Win=1024 + SSH-2.0-NmapNSE banner |
| Which service targeted? | SSH Port 22 | 2191 packets on Port 22 |
| How many attempts? | 3076 TCP connections | Statistics → Conversations |
| Was the attack successful? | Yes | Large encrypted packets after Key Exchange |
| When was attacker blocked? | T+390s | RST packets — fail2ban from Ch.02 |

---

## MITRE ATT&CK

| Technique | ID | Tool | Evidence in PCAP |
|:----------|:---|:-----|:-----------------|
| Active Scanning | T1595 | nmap -sS | SYN flood, Win=1024 |
| Active Scanning | T1595 | nmap -O | FIN+SYN+PSH+URG flags |
| Active Scanning | T1595 | nmap -sV | SSH-2.0-NmapNSE banner |
| Brute Force | T1110.001 | Hydra | Repeated Key Exchange pattern |
| Valid Accounts | T1078 | Hydra | Active encrypted session |

---

## Chapter Structure

| Folder | Content |
|:-------|:--------|
| [01-Architecture](01-Architecture/) | Lab environment, topology, capture setup |
| [02-Assets](02-Assets/) | Tools, PCAP info, Wireshark filters |
| [03-Attack_Scenario](03-Attack_Scenario/) | Attack commands + screenshots |
| [04-Evidences](04-Evidences/) | Wireshark screenshots + analysis |
| [05-LESSONS_LEARNED](05-LESSONS_LEARNED/) | Errors, solutions, key takeaways |

---

*homelab_AEGIS · github.com/cyb-ersin · Built from scratch, documented from zero.*
