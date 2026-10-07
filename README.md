# Project AEGIS

![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Series](https://img.shields.io/badge/Series-Project%20AEGIS-red)
![Type](https://img.shields.io/badge/Type-SOC%20%2F%20Detection%20Lab-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-red)
![IDS](https://img.shields.io/badge/IDS-Suricata-orange)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK-red)

---

## About

**Project AEGIS** is an advanced hands-on SOC and Detection Engineering home lab built on top of
[HomeLab_Foundation](https://github.com/cyb-ersin/HomeLab_Foundation).

The project started with a simple question:

> How much security visibility can I build from scratch inside my own lab?

Each chapter adds another part of the defensive workflow:

**traffic → telemetry → detection → correlation → investigation → response**

The focus is on understanding what the tools actually see, validating detections with evidence, documenting failures, and improving the lab step by step.

All offensive activity is performed only inside the owned and authorized lab environment.

---

## Final Lab Architecture

From **Chapter 04 onward**, AEGIS runs on a fixed three-host physical architecture.

```text
☠ h4des
ThinkPad X250
Kali Linux
Controlled Test Host
      │
      │ authorized lab traffic
      ▼
◈ edge
Fujitsu Lifebook E554
Ubuntu 24.04 LTS
Sensor / Monitored Host
      │
      ├── Suricata
      ├── Wazuh Agent
      └── Docker / OWASP Juice Shop
      │
      ▼
wazuh-server
Wazuh SIEM
VirtualBox VM on core
      │
      ▼
⌘ core
MacBook Pro
macOS
Management / Analysis / Hypervisor
```

---

## Chapters

| Chapter | Focus | Status |
|:--|:--|:--:|
| [Ch.01 — IDS Deployment](Ch01_IDS_Deployment/) | Suricata → Wazuh pipeline, reconnaissance and SSH brute-force detection | ✅ |
| [Ch.02 — Active Defense & Detection Engineering](Ch02_Active_Defense_Detection_Engineering/) | Wazuh Active Response, custom Suricata detection, fail2ban | ✅ |
| [Ch.03 — Network Forensics](Ch03_Network_Forensics/) | PCAP capture, Wireshark analysis, attack timeline reconstruction | ✅ |
| [Ch.04 — Web Application Attack, Detection & Incident Response](Ch04_Web_Application_Attack_Detection_Response/) | Burp, SQLi auth bypass, detection gap, custom detection, Wazuh, containment | ✅ |

---

## Key Outcomes

Project AEGIS demonstrated:

- Built a Suricata → Wazuh detection pipeline from scratch
- Detected and correlated network scanning and SSH brute-force activity
- Implemented automated response and custom detection logic
- Integrated fail2ban as an independent blocking layer
- Reconstructed an attack timeline from 8,936 captured packets
- Deployed OWASP Juice Shop as an intentionally vulnerable web target
- Used Burp Suite to inspect and replay authentication requests
- Performed a controlled SQL injection authentication bypass and validated administrative impact
- Identified a detection gap where malicious HTTP telemetry existed without a matching SQLi alert
- Wrote and validated custom Suricata rule SID `1000001`
- Confirmed the custom alert centrally in Wazuh
- Performed incident triage and source-specific containment
- Verified that the attacker was blocked while service availability remained intact for another client

These are validated milestones from completed AEGIS work.

---

## Final Workflow

```text
Attack simulation
      ↓
Network / host telemetry
      ↓
Suricata detection
      ↓
Wazuh correlation
      ↓
Investigation
      ↓
Containment
      ↓
Validation
```

---

## Tools Used

Suricata · Wazuh · Wireshark · tcpdump · Nmap · Hydra · fail2ban · iptables · Burp Suite · OWASP Juice Shop · Docker · curl

---

## Status

**Project AEGIS completed — October 2026.**

The next home-lab project will build on this foundation with a deeper enterprise / Active Directory purple-team environment.
