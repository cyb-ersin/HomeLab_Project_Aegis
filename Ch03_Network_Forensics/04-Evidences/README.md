# 04 — Evidences

## EV-01 — nmap SYN Scan Detected

**Filter:** `tcp.flags.syn == 1 && tcp.flags.ack == 0`  
**Result:** 3074 packets (34.4%)

![nmap SYN Scan](Bildschirmfoto%202026-04-11%20um%2013.19.56.png)

**What we see:**
- Source `192.168.178.129` (Kali) → all packets from the attacker
- `Win=1024` → nmap fingerprint! Normal traffic uses `Win=64240`
- SYN packets to hundreds of different ports in milliseconds → port scan confirmed

---

## EV-02 — Win=1024 vs Win=64240

![Win Size Comparison](Bildschirmfoto%202026-04-11%20um%2013.23.06.png)

Two distinct patterns side by side:

| Win Size | Tool | What it means |
|:---------|:-----|:--------------|
| `Win=1024` | nmap | Half-open scan — no real data transfer intended |
| `Win=64240` | Hydra | Real connection — data will be exchanged |

---

## EV-03 — SSH Traffic on Port 22

**Filter:** `tcp.port == 22 && tcp.flags.syn == 1`  
**Result:** 134 packets (1.5%)

![SSH SYN Packets](Bildschirmfoto%202026-04-11%20um%2013.29.15.png)

**What we see:**
- `Win=1024` packets → nmap probing Port 22
- `Win=64240` packets → Hydra building real SSH connections
- Red lines `[FIN, SYN, PSH, URG]` → nmap OS fingerprinting probe — impossible in normal traffic

---

## EV-04 — All SSH Traffic

**Filter:** `tcp.port == 22`  
**Result:** 2191 packets (24.5%)

![All SSH Traffic](Bildschirmfoto%202026-04-11%20um%2013.36.55.png)

Two sources visible:
- `192.168.178.122` (MacBook) → legitimate admin SSH session
- `192.168.178.129` (Kali) → attack traffic

tcpdump captures **everything** — including your own connections. Analyst context is required to separate them.

---

## EV-05 — MacBook SSH Session Identified

![MacBook SSH Session](Bildschirmfoto%202026-04-11%20um%2013.38.08.png)

`192.168.178.122` is the MacBook — not a threat. A SOC analyst must know the environment to separate legitimate from malicious traffic.

---

## EV-06 — nmap SSH Banner (Smoking Gun)

![nmap SSH Banner](Bildschirmfoto%202026-04-11%20um%2013.55.14.png)

```
Client: Protocol (SSH-1.5-NmapNSE-1.0)
Client: Protocol (SSH-1.5-Nmap-SSH1-Hostkey)
Server: Protocol (SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.15)
```

nmap identifies itself in **cleartext**. A SOC analyst seeing `NmapNSE` immediately knows: this is not a real SSH client.

---

## EV-07 — Attacker IP Isolated

**Filter:** `ip.addr == 192.168.178.129`  
**Result:** 7970 packets (89.2%)

![Attacker Traffic](Bildschirmfoto%202026-04-11%20um%2013.56.01.png)

89% of all captured traffic came from the attacker — clearly not normal behavior.

---

## EV-08 — OS Fingerprinting + RST Packets

![OS Fingerprinting and RST](Bildschirmfoto%202026-04-11%20um%2013.56.48.png)

**Red lines — RST packets:**
```
[RST] Win=0  → aegis-sentinel is blocking the connection (fail2ban from Ch.02!)
```

**ICMP lines:**
```
Echo request ttl=40  → nmap checks if host is alive
Echo reply   ttl=64  → host responds
```

**UDP line:**
```
ICMP Destination unreachable → port does not exist
```

---

## EV-09 — nmap SSH Stream Isolated

**Filter:** `tcp.stream eq 1017`

![nmap Stream](Bildschirmfoto%202026-04-11%20um%2014.08.48.png)

Complete nmap SSH connection in one view:
```
SYN          → connect request
SYN-ACK      → port is open!
Server Banner → OpenSSH 9.6p1 Ubuntu
Client Banner → SSH-2.0-NmapNSE ← attacker identified!
FIN          → immediately disconnects
```

nmap only needed the banner — connected and left in ~30ms.

---

## EV-10 — Follow Stream — Cipher Suites

![Follow Stream](Bildschirmfoto%202026-04-11%20um%2014.11.54.png)

nmap reads all supported cipher suites from the server — complete encryption configuration exposed during reconnaissance.

---

## EV-11 — Hydra Brute Force Pattern

**Filter:** `tcp.port == 22 && tcp.len > 100`  
**Result:** 123 packets (1.4%)

![Hydra Key Exchange Pattern](Bildschirmfoto%202026-04-11%20um%2014.33.01.png)

**Top section (repeating):**
```
Key Exchange Init (1042 bytes)  ← one password attempt
ECDH Reply (558 bytes)          ← attempt failed
Key Exchange Init (1042 bytes)  ← next attempt...
```

**Bottom section (unique):**
```
Encrypted packet (len=628)  ← large packet!
Encrypted packet (len=528)  ← active session!
```

→ Password was found — active SSH session established.

---

## EV-12 — Successful Login Confirmed

![Successful Login](Bildschirmfoto%202026-04-11%20um%2014.40.29.png)

After hundreds of short connections, suddenly a **long encrypted session** appears. Hydra is logged in. Provable from PCAP alone — no Hydra terminal output required.

---

## EV-13 — TCP Conversations — 3076 Total

**Menu:** Statistics → Conversations → TCP

![TCP Conversations](Bildschirmfoto%202026-04-11%20um%2014.49.03.png)

**3076 TCP connections — breakdown:**
- Many ports, 1 connection each → nmap port scan (T1595)
- Port 22, many short connections → Hydra brute force (T1110)
- Port 22, large data exchange → successful login (T1078)

---

## EV-14 — I/O Graph — Full Attack Timeline

**Menu:** Statistics → I/O Graph

![I/O Graph Timeline](Bildschirmfoto%202026-04-11%20um%2015.15.53.png)

```
T+60s   → Peak 1 (~2000 pkt/s) → nmap -sS    SYN Scan
T+100s  → Peak 2 (~2000 pkt/s) → nmap -O     OS Fingerprinting
T+120s  → Peak 3 (~2000 pkt/s) → nmap -sV    Service Detection
T+200s  → Peak 4 ( ~350 pkt/s) → Hydra       SSH Brute Force
T+390s  → Red zone (TCP Errors) → fail2ban    Kali blocked!
```

The entire attack timeline reconstructed — **no SIEM, no logs, packets only.**

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
