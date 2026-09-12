# Project AEGIS — Current Lab Architecture

This document defines the permanent hardware and logical architecture used by Project AEGIS from Chapter 04 onward.

Chapters 01–03 were completed in an earlier lab environment and remain unchanged as historical records of the systems actually used at that time.

---

## Architecture Overview

```mermaid
flowchart LR

    H["☠ h4des<br/>ThinkPad X250<br/>Kali Linux<br/>Controlled Test Host"]

    E["◈ edge<br/>Fujitsu Lifebook E554<br/>Ubuntu 24.04 LTS<br/>Sensor / Monitored Host"]

    W["wazuh-server<br/>Wazuh SIEM<br/>VirtualBox VM"]

    C[" core<br/>MacBook Pro<br/>macOS<br/>Management / Analysis / Hypervisor"]

    H -->|"Controlled lab traffic"| E
    E -->|"Security telemetry"| W
    W -->|"SOC visibility"| C
    C -->|"VirtualBox host"| W
