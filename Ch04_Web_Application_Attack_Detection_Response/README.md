# Ch.04 — Web Application Attack, Detection & Incident Response

**homelab_AEGIS** · SOC from Scratch · October 06, 2026

---

## Overview

Ch.04 extends AEGIS from network reconnaissance, brute-force detection and PCAP forensics into a web-application attack and incident-response workflow.

OWASP Juice Shop was deployed as an intentionally vulnerable target on `edge`. From `h4des`, Burp Suite was used to capture, inspect and replay the login request. A SQL injection payload bypassed authentication and created an administrative session.

Suricata already had visibility into the HTTP request, but the existing ruleset did not produce an SQL-injection alert for the successful bypass. This was treated as a **detection gap**.

A custom Suricata rule was then written and validated. The same request was repeated, the custom alert appeared in `eve.json`, and Wazuh received it centrally. The scenario was closed with incident triage and host-specific containment using `iptables`, while service availability for a legitimate client was preserved.

> **Attack → telemetry → detection gap → custom detection → Wazuh → investigation → containment**

---

## Lab Environment

| Machine | Role | OS / Platform | Final IP |
|:--------|:-----|:--------------|:---------|
| `☠ h4des` | Authorized attack / pentest host | Kali Linux | `192.168.178.50` |
| `◈ edge` | Web target + sensor + Wazuh agent | Ubuntu 24.04 LTS | `192.168.178.185` |
| `wazuh-server` | SIEM Manager / Indexer / Dashboard | Wazuh VM on `core` | `192.168.178.127` |
| `⌘ core` | Management + legitimate test client | macOS | DHCP |

**Target application:** OWASP Juice Shop · Docker · TCP/3000

---

## What Happened

```text
Kali / Burp
        ↓
POST /rest/user/login
        ↓
SQL Injection authentication bypass
        ↓
HTTP 200 + authenticated admin session
        ↓
Suricata sees HTTP traffic
        ↓
No matching SQLi alert
        ↓
Detection gap identified
        ↓
Custom Suricata SID 1000001
        ↓
Re-test → custom alert fires
        ↓
Wazuh receives alert
        ↓
Attacker IP contained with iptables
        ↓
Kali times out, core still receives HTTP 200
```

---

## What We Proved

| Question | Answer | Evidence |
|:---------|:-------|:---------|
| Was the normal invalid login rejected? | Yes | Burp → HTTP 401 |
| Could the login be bypassed with SQLi? | Yes | Burp → HTTP 200 |
| Was privileged access obtained? | Yes | Juice Shop Administration page |
| Did Suricata see the login traffic before custom detection? | Yes | HTTP events in `eve.json` |
| Did the existing ruleset detect this SQLi? | No | No matching SQLi alert |
| Could the gap be closed with a custom rule? | Yes | SID `1000001` fired |
| Did the alert reach the SIEM? | Yes | Wazuh rule `86601` |
| Could the attacking host be contained? | Yes | `DOCKER-USER` DROP rule |
| Did containment block Kali? | Yes | `curl` timeout |
| Did the service remain available to a legitimate host? | Yes | `core` → HTTP 200 |

---

## Detection Rule

```text
alert http any any -> 192.168.178.185 3000 (msg:"AEGIS SQLi Login Bypass Attempt"; flow:established,to_server; http.method; content:"POST"; http.uri; content:"/rest/user/login"; http.request_body; content:"admin@juice-sh.op'--"; classtype:web-application-attack; sid:1000001; rev:1;)
```

The source was intentionally changed from a fixed Kali IP to `any` after the lab hosts rebooted and DHCP changed the Kali address. The destination, endpoint and tested payload remained specific.

---

## Incident Response Summary

| Phase | Action | Result |
|:------|:-------|:-------|
| Triage | Identify source, target, endpoint, method and impact | Completed |
| Containment | Block `192.168.178.50` → TCP/3000 in `DOCKER-USER` | Completed |
| Validation | Re-test from Kali | Connection timed out |
| Service continuity | Test from `core` | HTTP 200 |
| Permanent remediation | Identify app-layer fix | Documented, not applied to intentionally vulnerable Juice Shop |

**Production remediation identified:** parameterized queries / prepared statements, secure authentication logic, server-side validation, least-privilege database permissions, token/session invalidation and post-fix security testing.

---

## Chapter Structure

| Folder | Content |
|:-------|:--------|
| [01-Architecture](01-Architecture/) | Current three-host topology + telemetry flow |
| [02-Assets](02-Assets/) | Tools, rule paths, Wazuh input, containment commands |
| [03-Attack_Scenario](03-Attack_Scenario/) | Baseline login, SQLi bypass, impact, detection gap |
| [04-Evidences](04-Evidences/) | Burp, Suricata, Wazuh and containment screenshots |
| [05-LESSONS_LEARNED](05-LESSONS_LEARNED/) | Errors, troubleshooting, IR takeaways |

---

## Project Closure

Ch.04 closes Project AEGIS.

The project progressed through:

1. IDS deployment and first detection
2. Active defense and detection engineering
3. Network forensics and PCAP analysis
4. Web exploitation, detection-gap analysis and incident containment

The final result is a complete lab story from **attack generation to centralized detection and response**.

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.04 — Web Application Attack, Detection & Incident Response*
