# homelab_AEGIS

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Series](https://img.shields.io/badge/Series-Project%20Aegis-red)
![IDS](https://img.shields.io/badge/IDS-Suricata-orange)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Certs](https://img.shields.io/badge/CompTIA-A%2B%20%7C%20Sec%2B-blue)

---

## About

`homelab_AEGIS` is the hands-on lab repository for **Project Aegis** — an advanced attack/defense series built on top of [homelab-foundation](https://github.com/cyb-ersin/homelab-foundation).

The scenario: a small organization with no existing security visibility. Starting from zero — no IDS, no SIEM, no incident response process. Each chapter builds one layer of the detection and response pipeline until the environment is fully instrumented.

> **homelab-foundation** taught the tools.  
> **homelab_AEGIS** puts them to work.

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

Complete [homelab-foundation](https://github.com/cyb-ersin/homelab-foundation) first:

- ✅ Lab 0 — Network Discovery
- ✅ Lab 1 — Wireshark Traffic Analysis
- ✅ Lab 2 — WiFi Security
- ✅ Lab 3 — Firewall & Segmentation

---

## Chapters

| # | Chapter | Scenario Question | Status |
|---|---|---|---|
| 01 | [IDS Deployment](Ch01_IDS_Deployment/) | Can we detect a port scan and brute force in real time? | 🔜 |
| 02 | [SIEM Integration](Ch02_SIEM_Integration/) | Do Suricata alerts reach Wazuh? Can we build a dashboard? | 🔜 |
| 03 | [Exploitation & Detection](Ch03_Exploitation/) | If Metasploit gets a shell — does the IDS see it? | 🔜 |
| 04 | [Lateral Movement](Ch04_Lateral_Movement/) | Can SIEM detect movement between hosts post-compromise? | 🔜 |
| 05 | [PCAP Forensics](Ch05_PCAP_Forensics/) | What does C2 and infostealer traffic look like in a PCAP? | 🔜 |
| 06 | [Detection Rule Writing](Ch06_Rule_Writing/) | Can we write a Suricata rule that catches a specific threat? | 🔜 |
| 07 | [Incident Response](Ch07_Incident_Response/) | Can we build a full IR timeline and harden the environment? | 🔜 |

---

## Lab Environment

| Machine | OS | Role |
|---|---|---|
| iMac 12,1 (32GB RAM) | Ubuntu | Primary server — Suricata + Wazuh |
| ThinkPad X250 | Kali Linux | Attacker |
| MacBook Pro | macOS | Management / documentation |
| Ubuntu Laptop | Ubuntu | Secondary target |
| Fritz!Box | — | Network gateway |

---

## Roadmap

```
homelab-foundation              ✅ complete
└── Scanning · Traffic analysis · WiFi · Firewall

homelab_AEGIS                   ← you are here
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

## Privacy & Ethics

- All attack simulations performed on own lab network only
- No third-party systems targeted at any stage
- PCAP forensics performed on publicly available malware samples
- No credentials or sensitive data included in documentation

---

*Started from zero. Still learning, still building.*  
*[homelab-foundation](https://github.com/cyb-ersin/homelab-foundation) → homelab_AEGIS → ...*
