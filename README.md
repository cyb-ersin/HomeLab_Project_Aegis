# Project Aegis

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![Type](https://img.shields.io/badge/Type-Attack%20%2F%20Defense-red)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-red)
![IDS](https://img.shields.io/badge/IDS-Suricata-orange)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Prereq](https://img.shields.io/badge/Prereq-homelab--foundation-lightgrey)

---

## About

Project Aegis is an advanced attack/defense lab series building directly on [homelab-foundation](https://github.com/cyb-ersin/homelab-foundation).

The series follows a fictional but realistic scenario: a **small company's SOC is built from scratch** — no existing visibility, no detection capability. Each chapter adds a layer until a full detection and response pipeline is operational.

> Every lab answers a single, measurable question.  
> Every finding is backed by log evidence.  
> Every detection maps to MITRE ATT&CK.

---

## Scenario

> *A small organization has a flat network, no IDS, no SIEM, and no incident response process. An attacker has already gained initial foothold. The SOC team must build visibility, detect the intrusion, investigate it, and harden the environment — in real time.*

---

## Prerequisites

Before starting this series, complete [homelab-foundation](https://github.com/cyb-ersin/homelab-foundation):

- ✅ Lab 0 — Network Discovery
- ✅ Lab 1 — Wireshark Traffic Analysis
- ✅ Lab 2 — WiFi Security
- ✅ Lab 3 — Firewall & Segmentation

---

## Stack

| Role | Tool | Why |
|---|---|---|
| IDS/IPS | Suricata | Multi-threaded, JSON EVE logs, native SIEM integration |
| SIEM | Wazuh | Open-source, active community, Suricata native support |
| Rule source | Emerging Threats (ET) | Industry standard, updated daily |
| Attacker | Kali Linux | Metasploit, hydra, nmap, custom payloads |
| PCAP analysis | Wireshark + Zeek | Forensics and IOC extraction |

---

## Lab Series

| # | Chapter | Scenario Question | Status |
|---|---|---|---|
| 01 | IDS Deployment | Can we detect a port scan and brute force in real time? | 🔜 |
| 02 | SIEM Integration | Do Suricata alerts reach Wazuh? Can we build a dashboard? | 🔜 |
| 03 | Exploitation & Detection | If Metasploit gets a shell — does the IDS see it? | 🔜 |
| 04 | Lateral Movement | Can SIEM detect movement between hosts post-compromise? | 🔜 |
| 05 | PCAP Forensics | What does C2 and infostealer traffic look like in a PCAP? | 🔜 |
| 06 | Detection Rule Writing | Can we write a Suricata rule that catches a specific threat? | 🔜 |
| 07 | Incident Response | Can we build a full IR timeline and harden the environment? | 🔜 |

---

## Lab Environment

| Machine | OS | Role |
|---|---|---|
| iMac 12,1 (32GB RAM) | Ubuntu | Primary lab server — Suricata + Wazuh host |
| ThinkPad X250 | Kali Linux | Attacker |
| MacBook Pro | macOS | Management / documentation |
| MacBook Pro | Ubuntu VM | Secondary target |
| Fritz!Box | — | Network gateway |

---

## Roadmap

```
homelab-foundation              ✅ complete
└── Foundations: scanning, traffic analysis,
    WiFi security, firewall & segmentation

project-aegis                   ← you are here
├── Ch.01  IDS Deployment        🔜
├── Ch.02  SIEM Integration      🔜
├── Ch.03  Exploitation          🔜
├── Ch.04  Lateral Movement      🔜
├── Ch.05  PCAP Forensics        🔜
├── Ch.06  Rule Writing          🔜
└── Ch.07  Incident Response     🔜
```

---

## MITRE ATT&CK Coverage (Planned)

| Tactic | Technique | ID |
|---|---|---|
| Reconnaissance | Network Service Discovery | T1046 |
| Initial Access | Exploit Public-Facing Application | T1190 |
| Execution | Command and Scripting Interpreter | T1059 |
| Lateral Movement | Remote Services | T1021 |
| Collection | Network Sniffing | T1040 |
| Defense Evasion | Impair Defenses | T1562 |
| Command & Control | Application Layer Protocol | T1071 |
| Exfiltration | Exfiltration Over C2 Channel | T1041 |

---

*Part of an ongoing homelab series. Building toward Junior SOC Analyst.*
