# 03 — Attack Scenario

## Overview

A controlled SSH brute force attack was launched from the Kali laptop against aegis-sentinel. The attack was used to validate all three defense layers: Wazuh Active Response, Custom Suricata Rule, and fail2ban.

---

## Attacker Profile

| Field | Value |
|:------|:------|
| Machine | Kali Laptop (physical) |
| IP | 172.20.10.3 |
| Tool | Hydra v9.6 |
| Target | aegis-sentinel — 172.20.10.6:22 |
| Wordlist | /usr/share/wordlists/rockyou.txt |
| Threads | 4 |

---

## Attack Command

```bash
hydra -l aegis-siem -P /usr/share/wordlists/rockyou.txt -t 4 ssh://172.20.10.6
```

### Flag Breakdown

| Flag | Value | Description |
|:-----|:------|:------------|
| `-l` | aegis-siem | Single username (lowercase = single value) |
| `-P` | rockyou.txt | Password list (uppercase = list) |
| `-t` | 4 | Parallel threads |
| `ssh://` | protocol | Target service |
| `172.20.10.6` | target IP | aegis-sentinel |

![Hydra Attack](Bildschirmfoto%202026-03-24%20um%2009.53.44.png)

---

## Attack Timeline

| Time | Event |
|:-----|:------|
| 09:39 | Hydra brute force started |
| 09:41 | Wazuh Active Response triggered — iptables DROP |
| 09:46 | Active Response timeout (300s) — IP unblocked |
| 09:53 | Second attack — IP blocked again |

---

## MITRE ATT&CK Mapping

| Phase | Technique | ID | Tool |
|:------|:----------|:---|:-----|
| Reconnaissance | Active Scanning | T1595 | nmap |
| Credential Access | Brute Force: Password Guessing | T1110.001 | Hydra + rockyou.txt |
| Lateral Movement | Remote Services: SSH | T1021.004 | Hydra SSH |
| Initial Access | Valid Accounts | T1078 | SSH with stolen credentials |
| Privilege Escalation | Sudo and Sudo Caching | T1548.003 | sudo commands |
