# Project AEGIS

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
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

The goal is not simply to run attacks or collect screenshots.

Each chapter builds another part of the defensive workflow:

**traffic → telemetry → detection → correlation → investigation → response**

The focus is on understanding what the tools actually see, validating detections with evidence, documenting failures, and improving the lab step by step.

---

## Scenario

Project AEGIS simulates a small environment in which security visibility is gradually built from the ground up.

The defender must be able to:

- observe suspicious network activity
- collect and centralize telemetry
- detect reconnaissance and authentication attacks
- correlate events in a SIEM
- reconstruct activity from packet evidence
- write and validate detection rules
- investigate exploitation and post-compromise behavior
- document detection gaps and hardening opportunities

All offensive activity is performed only inside the owned and authorized lab environment.

---

## Current Lab Architecture

From **Chapter 04 onward**, AEGIS runs on a fixed three-host physical architecture.

```text
☠ h4des
ThinkPad X250
Kali Linux
Controlled Test Host
      │
      │ controlled lab traffic
      ▼
◈ edge
Fujitsu Lifebook E554
Ubuntu 24.04 LTS
Sensor / Monitored Host
      │
      │ security telemetry
      ▼
wazuh-server
Wazuh SIEM
VirtualBox VM on core
      │
      │ centralized analysis
      ▼
 core
MacBook Pro
macOS
Management / Analysis / Hypervisor
```
---

---

## Key Outcomes

So far, Project AEGIS has demonstrated:

- Built a Suricata → Wazuh detection pipeline from scratch
- Detected and correlated network scanning and SSH brute-force activity
- Implemented automated response and custom detection logic
- Reconstructed an attack timeline from 8,936 captured packets
- Established a permanent three-host architecture for continued SOC and Detection Engineering work

These are validated milestones from completed AEGIS work, not planned capabilities.
