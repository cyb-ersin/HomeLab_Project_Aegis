# Project AEGIS

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Series](https://img.shields.io/badge/Series-Project%20AEGIS-red)
![Focus](https://img.shields.io/badge/Focus-SOC%20%7C%20Detection%20Engineering-blue)
![IDS](https://img.shields.io/badge/IDS-Suricata-orange)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK-red)

---

## About

**Project AEGIS** is my hands-on SOC and Detection Engineering home lab.

The project builds on the networking and security fundamentals developed in
[HomeLab Foundation](https://github.com/cyb-ersin/HomeLab_Foundation).

The goal is not simply to generate attacks or collect screenshots.

The goal is to understand the complete defensive workflow:

**traffic → telemetry → detection → correlation → investigation → response**

Each chapter adds another layer to that process.

---

## Current Architecture

From Chapter 04 onward, Project AEGIS runs on a fixed three-host physical architecture.

```text
                     Project AEGIS

      ☠ h4des
      ThinkPad X250
      Kali Linux
      Controlled Test Host
            │
            │ controlled security testing
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
      VirtualBox VM
            │
            │ centralized analysis
            ▼
       core
      MacBook Pro
      macOS
      Management / Analysis / Hypervisor
