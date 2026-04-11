# 04 — Evidences

## Evidence Overview

| # | Screenshot | Filter / Tool | What it shows |
|:--|:-----------|:-------------|:--------------|
| 01 | tcpdump-capture.png | aegis-sentinel terminal | 8936 packets captured, 0 dropped |
| 02 | nmap-sS-output.png | Kali terminal | nmap -sS -A -T4 result — Port 22 open |
| 03 | nmap-sV-output.png | Kali terminal | nmap -sV result — OpenSSH 9.6p1 |
| 04 | nmap-O-output.png | Kali terminal | nmap -O result — Linux OS detected |
| 05 | hydra-success.png | Kali terminal | Hydra — password 1111 found |
| 06 | ws-syn-scan.png | tcp.flags.syn==1 && tcp.flags.ack==0 | 3074 SYN packets, Win=1024 nmap fingerprint |
| 07 | ws-syn-scan-patterns.png | tcp.flags.syn==1 && tcp.flags.ack==0 | Win=1024 vs Win=64240 comparison |
| 08 | ws-port22-syn.png | tcp.port==22 && tcp.flags.syn==1 | 134 packets — nmap + Hydra on Port 22 |
| 09 | ws-port22-all.png | tcp.port==22 | 2191 packets — MacBook SSH + attack traffic |
| 10 | ws-nmap-banner.png | tcp.port==22 | SSH-2.0-NmapNSE banner in cleartext |
| 11 | ws-attacker-ip.png | ip.addr==192.168.178.129 | 7970 packets (89.2%) from attacker |
| 12 | ws-os-fingerprint.png | ip.addr==192.168.178.129 | ICMP probes + UDP + FIN/SYN/PSH/URG flags |
| 13 | ws-stream-1017.png | tcp.stream eq 1017 | nmap SSH stream isolated |
| 14 | ws-cipher-suites.png | Follow Stream 1017 | SSH-2.0-NmapNSE + full cipher suite list |
| 15 | ws-hydra-pattern.png | tcp.port==22 && tcp.len>100 | 123 packets — Key Exchange pattern |
| 16 | ws-successful-login.png | ip.addr==192.168.178.129 | Diffie-Hellman + large encrypted packets |
| 17 | ws-rst-packets.png | ip.addr==192.168.178.129 | RST packets — fail2ban blocking Kali |
| 18 | ws-conversations.png | Statistics → Conversations | 3076 TCP conversations |
| 19 | ws-io-graph.png | Statistics → I/O Graph | 4 attack peaks + red TCP errors at T+390s |

---

## Evidence Details

### EV-01 — tcpdump Capture Confirmation
**Tool:** aegis-sentinel terminal  
**Shows:** `8936 packets captured · 8945 received · 0 dropped by kernel`  
**Significance:** Confirms complete capture with no packet loss — forensic integrity verified.

---

### EV-02/03/04 — nmap Attack Output
**Tool:** Kali terminal  
**Shows:** Three nmap scans with results — Port 22 open, OpenSSH 9.6p1, Linux OS detected  
**Significance:** The attacker's perspective — what information was gathered before brute force.

---

### EV-05 — Hydra Success
**Tool:** Kali terminal  
**Shows:** `[22][ssh] host: 192.168.178.126 login: aegis-siem password: 1111`  
**Significance:** Confirms successful credential compromise — T1078 Initial Access.

---

### EV-06/07 — nmap SYN Scan in Wireshark
**Filter:** `tcp.flags.syn == 1 && tcp.flags.ack == 0`  
**Shows:** 3074 SYN packets all from 192.168.178.129  
**Key indicator:** `Win=1024` — nmap's distinctive window size fingerprint  
**Normal traffic:** `Win=64240` (Hydra/browsers use standard window sizes)

---

### EV-08 — SSH Traffic Split
**Filter:** `tcp.port == 22 && tcp.flags.syn == 1`  
**Shows:** 134 packets — mixture of nmap (Win=1024) and Hydra (Win=64240) on Port 22  
**Also visible:** `[FIN, SYN, PSH, URG]` — nmap `-O` OS fingerprinting probe (impossible in normal traffic)

---

### EV-09 — MacBook vs Attacker Traffic
**Filter:** `tcp.port == 22`  
**Shows:** 2191 packets — MacBook SSH session (192.168.178.122) alongside attack traffic  
**Key learning:** tcpdump captures ALL traffic including legitimate admin sessions — context required for analysis

---

### EV-10 — nmap SSH Banner (Smoking Gun)
**Filter:** `tcp.port == 22`  
**Shows:**
```
Client: Protocol (SSH-1.5-NmapNSE-1.0)
Client: Protocol (SSH-1.5-Nmap-SSH1-Hostkey)
Server: Protocol (SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.15)
```
**Significance:** nmap identifies itself in cleartext — immediate attacker identification without any other context.

---

### EV-11/12 — Attacker IP Isolation
**Filter:** `ip.addr == 192.168.178.129`  
**Shows:** 7970 packets (89.2% of total capture)  
**Breakdown visible:**
- Many ports, 1 SYN each → nmap port scan
- Port 22, repeated Key Exchange → Hydra brute force
- Port 22, large encrypted packets → successful login

---

### EV-13 — nmap Stream Isolated
**Filter:** `tcp.stream eq 1017`  
**Shows:** Complete nmap SSH handshake sequence:
```
SYN → SYN-ACK → ACK → Server Banner → Client Banner → Key Exchange → FIN
```
**Duration:** ~30ms — nmap disconnects immediately after fingerprinting

---

### EV-14 — Full Cipher Suite (Follow Stream)
**Tool:** Follow TCP Stream on Stream 1017  
**Shows:** `SSH-2.0-OpenSSH_9.6p1` + full cipher suite negotiation list  
**Significance:** Complete server capability disclosure visible to attacker

---

### EV-15 — Hydra Key Exchange Pattern
**Filter:** `tcp.port == 22 && tcp.len > 100`  
**Shows:** 123 packets — repeated Key Exchange Init sequences  
**Pattern:**
```
Client: Key Exchange Init (1042 bytes)   ← one password attempt
Server: Key Exchange Init (1186 bytes)
Server: ECDH Reply (558 bytes)
→ connection fails → new Key Exchange starts
→ repeated hundreds of times
```

---

### EV-16 — Successful Login Evidence
**Filter:** `ip.addr == 192.168.178.129`  
**Shows:** After hundreds of failed attempts:
```
Server: Encrypted packet (len=628)   ← large packets = active session
Client: Encrypted packet (len=528)
Server: Encrypted packet (len=548)
```
**Significance:** Packet size shift proves successful login — identifiable without Hydra terminal output.

---

### EV-17 — fail2ban RST Packets
**Filter:** `ip.addr == 192.168.178.129`  
**Shows:** Red RST packets from aegis-sentinel at T~390s:
```
22 → [RST] Seq=1260 Win=0 Len=0
```
**Significance:** Ch.02 defense (fail2ban + Active Response) still active — visible cross-chapter impact.

---

### EV-18 — Conversations (3076 TCP)
**Tool:** Statistics → Conversations → TCP  
**Shows:** 3076 TCP conversations total  
**Pattern:**
- One conversation per port → nmap scan
- Multiple conversations on Port 22 → Hydra brute force
- One long conversation on Port 22 with large data → successful login

---

### EV-19 — I/O Graph Timeline
**Tool:** Statistics → I/O Graph  
**Shows:**
```
T+60s   → Peak 1 (~2000 pkt/s)   → nmap -sS    SYN Scan
T+100s  → Peak 2 (~2000 pkt/s)   → nmap -O     OS Fingerprinting
T+120s  → Peak 3 (~2000 pkt/s)   → nmap -sV    Service Detection
T+200s  → Peak 4 (~350 pkt/s)    → Hydra       SSH Brute Force
T+390s  → Red TCP Errors zone     → fail2ban    Kali IP Blocked
```
**Significance:** Full attack timeline reconstructed from PCAP — no SIEM required.

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
