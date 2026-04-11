# 03 — Attack Scenario

## Overview

Four attacks were executed from Kali (192.168.178.129) against aegis-sentinel (192.168.178.126) while `tcpdump` captured all traffic.

---

## Attack 1 — nmap SYN Scan

```bash
sudo nmap -sS -A -T4 192.168.178.126
```

| Flag | Meaning |
|:-----|:--------|
| `-sS` | SYN scan — sends SYN only, never completes TCP handshake |
| `-A` | Aggressive — OS detection, version detection, traceroute |
| `-T4` | Fast timing |

**Result:**

![nmap SYN Scan Output](1.png)

```
PORT   STATE  SERVICE  VERSION
22/tcp open   ssh      OpenSSH 9.6p1 Ubuntu
OS: Linux 4.15-5.19
```

**MITRE:** T1595 — Active Scanning

---

## Attack 2 — nmap OS Fingerprinting

```bash
sudo nmap -O 192.168.178.126
```

**Result:**

![nmap OS Fingerprinting Output](4.png)

**Key forensic artifact:** nmap sends impossible flag combinations (`FIN+SYN+PSH+URG`) to fingerprint the OS — immediately visible in PCAP.

**MITRE:** T1595 — Active Scanning

---

## Attack 3 — nmap Service Version Detection

```bash
sudo nmap -sV 192.168.178.126
```

**Result:**

![nmap Service Detection Output](5.png)

```
22/tcp open ssh OpenSSH 9.6p1 Ubuntu 3ubuntu13.15
```

**Key forensic artifact:** nmap reads the SSH banner and identifies itself as `SSH-2.0-NmapNSE` — visible in cleartext inside the PCAP.

**MITRE:** T1595 — Active Scanning

---

## Attack 4 — Hydra SSH Brute Force

```bash
hydra -l aegis-siem -P /tmp/rockyou_9800.txt ssh://192.168.178.126
```

**Result:**

![Hydra Brute Force Success](3.png)

```
[22][ssh] host: 192.168.178.126   login: aegis-siem   password: 1111 ✅
1 of 1 target successfully completed
```

**Note:** fail2ban from Ch.02 was still active — blocked Kali after 5 attempts. Required reset before running:

```bash
sudo iptables -F
sudo fail2ban-client unban --all
```

**MITRE:** T1110.001 → T1078

---

## MITRE ATT&CK Kill Chain

| Phase | Technique | ID | Tool |
|:------|:----------|:---|:-----|
| Reconnaissance | Active Scanning | T1595 | nmap -sS |
| Reconnaissance | Active Scanning | T1595 | nmap -O |
| Reconnaissance | Active Scanning | T1595 | nmap -sV |
| Credential Access | Brute Force | T1110.001 | Hydra |
| Initial Access | Valid Accounts | T1078 | Hydra |

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
