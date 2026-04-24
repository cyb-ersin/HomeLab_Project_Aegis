# Project Aegis

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Series](https://img.shields.io/badge/Series-Project%20Aegis-red)
![Type](https://img.shields.io/badge/Type-Attack%20%2F%20Defense-red)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-red)
![IDS](https://img.shields.io/badge/IDS-Suricata-orange)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Certs](https://img.shields.io/badge/CompTIA-A%2B%20%7C%20Sec%2B-blue)

---

## About

**Project Aegis** is an advanced attack/defense lab series — an advanced attack/defense series built on top of [HomeLab_Foundation](https://github.com/cyb-ersin/HomeLab_Foundation).

The scenario: a small organization with no existing security visibility. Starting from zero — no IDS, no SIEM, no incident response process. Each chapter builds one layer of the detection and response pipeline until the environment is fully instrumented.

> **homelab-foundation** taught the tools.  
> **Project Aegis** puts them to work.

---

## Scenario

> *An attacker has gained initial foothold in a flat network. The defender must build visibility from scratch — deploy IDS, integrate SIEM, detect active exploitation, investigate lateral movement, analyze forensic artifacts, and harden the environment. All in real time. All documented.*

---

## Stack

| Role | Tool |
|---|---|
| IDS/IPS | Suricata |
| SIEM | Wazuh |
| Rule source | Emerging Threats (ET Open) |
| Attacker | Kali Linux — Metasploit, hydra, nmap |
| Forensics | Wireshark, Zeek |
| Documentation | Markdown + MITRE ATT&CK mapping |

---

## Prerequisites

Complete [HomeLab_Foundation](https://github.com/cyb-ersin/HomeLab_Foundation)first:

- ✅ Lab 0 — Network Discovery
- ✅ Lab 1 — Wireshark Traffic Analysis
- ✅ Lab 2 — WiFi Security
- ✅ Lab 3 — Firewall & Segmentation

---

## Chapters


| # | Chapter | Scenario Question | Status |
|:--|:--------|:------------------|:-------|
| 01 | [IDS Deployment & First Detection](https://github.com/cyb-ersin/HomeLab_Project_Aegis/tree/main/Ch01_IDS_Deployment) | Can we detect a port scan and brute force in real time? | ✅ Complete |
| 02 | [Active Defense & Detection Engineering](https://github.com/cyb-ersin/HomeLab_Project_Aegis/tree/main/Ch02_Active_Defense_Detection_Engineering)| Can we automatically block an attacker — and detect it in the SIEM? | ✅ Complete |
| 03 | [Network Forensics & PCAP Analysis](https://github.com/cyb-ersin/HomeLab_Project_Aegis/tree/main/Ch03_Network_Forensics) | What does the attack look like on packet level? Can we reconstruct the timeline? | ✅ Complete  |
| 04 | Exploitation & Detection | If Metasploit gets a shell — does the IDS see it? | 🔜 Planned |
| 05 | Lateral Movement | Can SIEM detect movement between hosts post-compromise? | 🔜 Planned |
| 06 | Detection Rule Writing | Can we write a Suricata rule that catches a specific threat? | 🔜 Planned |
| 07 | Incident Response | Can we build a full IR timeline and harden the environment? | 🔜 Planned |

---

## Lab Environment

| Machine | OS | Role |
|---|---|---|
| iMac 12,1 (32GB RAM) | Ubuntu | Primary server — Suricata + Wazuh |
| ThinkPad X250 | Kali Linux | Attacker |
| MacBook Pro | macOS | Management / documentation |
| MacBook Pro | Ubuntu VM | Secondary target |
| Fritz!Box | — | Network gateway |

---

## Roadmap

```
homelab-foundation              ✅ complete
└── Scanning · Traffic analysis · WiFi · Firewall

homelab_AEGIS                   ← you are here
├── Ch.01  IDS Deployment & First Detection     ✅
├── Ch.02  Active Defense & Detection Eng.      ✅
├── Ch.03  Network Forensics & PCAP Analysis    ✅
├── Ch.04  Exploitation & Detection             🔄 (jetzt)
├── Ch.05  Lateral Movement                     🔜
├── Ch.06  Detection Rule Writing               🔜
└── Ch.07  Incident Response                    🔜
```

---

## MITRE ATT&CK Coverage

| Tactic | Technique | ID | Status |
|---|---|---|---|
| Reconnaissance | Active Scanning | T1595 | ✅ Ch.01 |
| Credential Access | Brute Force: Password Guessing | T1110.001 | ✅ Ch.01/02 |
| Lateral Movement | Remote Services: SSH | T1021.004 | ✅ Ch.01/02 |
| Initial Access | Valid Accounts | T1078 | ✅ Ch.01/02 |
| Privilege Escalation | Sudo and Sudo Caching | T1548.003 | ✅ Ch.02 |
| Collection | Network Sniffing | T1040 | ✅ Ch.03 |
| Initial Access | Exploit Public-Facing Application | T1190 | 🔜 Ch.04 |
| Execution | Command and Scripting Interpreter | T1059 | 🔜 Ch.04 |
| Defense Evasion | Impair Defenses | T1562 | 🔜 Ch.06 |
| Command & Control | Application Layer Protocol | T1071 | 🔜 Ch.05 |
| Exfiltration | Exfiltration Over C2 Channel | T1041 | 🔜 Ch.05 |

---

## Privacy & Ethics

- All attack simulations performed on own lab network only
- No third-party systems targeted at any stage
- PCAP forensics performed on publicly available samples only
- No credentials or sensitive data included in documentation

---

*Started from zero. Still learning, still building.*  
*[homelab-foundation](https://github.com/cyb-ersin/HomeLab_Foundation) → homelab_AEGIS → ...*
