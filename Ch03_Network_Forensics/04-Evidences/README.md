# 04 — Evidences

## EV-01 — tcpdump Capture Confirmation

**Machine:** aegis-sentinel terminal

![tcpdump capture](Bildschirmfoto%202026-04-11%20um%2012.42.20.png)

```
tcpdump: listening on enp0s3, link-type EN10MB
8936 packets captured · 8945 received · 0 dropped by kernel
```

Zero packet loss — forensic integrity confirmed. All attack traffic was recorded.

---

## EV-02 — nmap SYN Scan — 3074 Packets Detected

**Filter:** `tcp.flags.syn == 1 && tcp.flags.ack == 0`

![SYN scan 3074 packets](Bildschirmfoto%202026-04-11%20um%2013.19.56.png)

3074 SYN packets, all from `192.168.178.129` (Kali) to `192.168.178.126` (aegis-sentinel). All show `Win=1024` — the nmap fingerprint. Normal traffic uses `Win=64240`.

---

## EV-03 — nmap Fingerprint: Win=1024 vs Win=64240

**Filter:** `tcp.flags.syn == 1 && tcp.flags.ack == 0`

![Win size fingerprint](Bildschirmfoto%202026-04-11%20um%2013.23.06.png)

Two tools clearly visible:
- `Win=1024` → nmap half-open scan packets
- `Win=64240` → Hydra real SSH connection attempts
- Red lines `[FIN, SYN, PSH, URG]` → nmap `-O` OS fingerprinting probe — this flag combination is physically impossible in normal traffic

---

## EV-04 — SSH Port 22 SYN Packets — 134 Packets

**Filter:** `tcp.port == 22 && tcp.flags.syn == 1`

![SSH SYN 134 packets](Bildschirmfoto%202026-04-11%20um%2013.29.15.png)

134 packets (1.5%) — only Port 22 SYN traffic. Both nmap (`Win=1024`) and Hydra (`Win=64240`) visible in the same view. Red RST packets = fail2ban blocking Kali.

---

## EV-05 — MacBook Admin Session Identified

**Filter:** `tcp.port == 22`

![MacBook SSH session](Bildschirmfoto%202026-04-11%20um%2013.36.55.png)

All traffic here comes from `192.168.178.122` — the MacBook admin SSH session. Completely legitimate. tcpdump records everything including admin connections. A SOC analyst must separate this from attack traffic using context.

---

## EV-06 — nmap Banner + RST + Hydra in One View

**Filter:** `tcp.port == 22`

![nmap banner RST Hydra](Bildschirmfoto%202026-04-11%20um%2013.38.08.png)

Three things visible at once:
- **Red RST lines** → fail2ban blocking Kali's connections
- **Lines 2394–2395:** `SSH-1.5-NmapNSE-1.0` and `SSH-1.5-Nmap-SSH1-Hostkey` → nmap identifies itself in cleartext
- **`Win=64240` SYN packets (lines 2382+)** → Hydra starting brute force after nmap scan

---

## EV-07 — Attacker Traffic Isolated — SYN Scan Pattern

**Filter:** `ip.addr == 192.168.178.129`

![Attacker SYN scan](Bildschirmfoto%202026-04-11%20um%2013.55.14.png)

7970 packets (89.2%) from the attacker. SYN packets to hundreds of different ports with `Win=1024` — classic nmap port scan. One connection per port, rapid succession.

---

## EV-08 — OS Fingerprinting Artifacts

**Filter:** `ip.addr == 192.168.178.129`

![OS fingerprinting](Bildschirmfoto%202026-04-11%20um%2013.55.51.png)

- **Red RST lines** → fail2ban blocking connections
- **ICMP Echo request/reply** → nmap host discovery ping
- **UDP line → ICMP Destination unreachable** → nmap probing closed UDP port
- **`[FIN, SYN, PSH, URG]`** → nmap `-O` OS fingerprinting probe — impossible in normal traffic, unique nmap signature

---

## EV-09 — nmap SSH Service Detection Banner

**Filter:** `ip.addr == 192.168.178.129`

![nmap SSH service banner](Bildschirmfoto%202026-04-11%20um%2013.56.01.png)

```
Client: Protocol (SSH-2.0-Nmap-SSH2-Hostkey)
Server: Protocol (SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.15)
```

nmap `-sV` reads the full server banner. Server version `OpenSSH 9.6p1 Ubuntu` fully exposed — attacker now knows exact version for vulnerability research.

---

## EV-10 — Successful Login via Encrypted Packet Size

**Filter:** `ip.addr == 192.168.178.129`

![Successful login](Bildschirmfoto%202026-04-11%20um%2013.56.31.png)

After hundreds of failed short attempts, a persistent encrypted session appears:
```
Server: Diffie-Hellman Key Exchange Reply, New Keys (len=200)
Client: Encrypted packet (len=44)
Server: Encrypted packet (len=44)
Server: Encrypted packet (len=264)
```

Ongoing bidirectional encrypted session = Hydra found password `1111`. Proven from packet behavior alone — no Hydra terminal output needed.

---

## EV-11 — fail2ban Blocking at T+390s

**Filter:** `ip.addr == 192.168.178.129`

![fail2ban RST blocking](Bildschirmfoto%202026-04-11%20um%2013.56.48.png)

```
46158 → 22 [RST] Seq=1260 Win=0 Len=0
46188 → 22 [RST] Seq=1260 Win=0 Len=0
ICMP: Destination unreachable (Port unreachable)
```

fail2ban from Ch.02 still active — blocking Kali at packet level. Cross-chapter defense-in-depth confirmed in the PCAP.

---

## EV-12 — nmap SSH Stream Isolated

**Filter:** `tcp.stream eq 1017`

![nmap stream isolated](Bildschirmfoto%202026-04-11%20um%2014.08.48.png)

Complete nmap SSH connection in one view:
```
SYN → SYN-ACK → ACK
Server: OpenSSH_9.6p1 Ubuntu banner
Client: SSH-2.0-NmapNSE ← tool identified
Client: Key Exchange Init
Server: Key Exchange Init
FIN-ACK → immediate disconnect
```

nmap grabbed the banner and disconnected in ~30ms. No authentication attempted.

---

## EV-13 — Follow Stream — Cipher Suites Exposed

**Filter:** `tcp.stream eq 1017` → Follow TCP Stream

![Follow stream cipher suites](Bildschirmfoto%202026-04-11%20um%2014.11.54.png)

```
SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.15
SSH-2.0-Nmap-SSH2-Hostkey
[full cipher suite list...]
```

Complete server encryption capabilities exposed to attacker during reconnaissance phase.

---

## EV-14 — Hydra Brute Force Pattern + Successful Login

**Filter:** `tcp.port == 22 && tcp.len > 100`

![Hydra key exchange pattern](Bildschirmfoto%202026-04-11%20um%2014.33.01.png)

**Top (repeating — failed attempts):**
```
Client: Key Exchange Init (1042 bytes)
Server: Key Exchange Init (1186 bytes)
Server: ECDH Reply (558 bytes) → failed → next attempt
```

**Bottom (unique — successful login):**
```
Server: Encrypted packet (len=628)
Client: Encrypted packet (len=528)
Server: Encrypted packet (len=548)
```

Password found — active SSH session established.

---

## EV-15 — Conversations: MacBook Only (with filter)

**Menu:** Statistics → Conversations → TCP (display filter active)

![Conversations MacBook only](Bildschirmfoto%202026-04-11%20um%2014.40.29.png)

With filter applied: only 1 conversation visible — the MacBook admin SSH session (`192.168.178.122`). Shows how filters isolate specific traffic for clean analysis.

---

## EV-16 — Conversations: 3076 TCP Total (no filter)

**Menu:** Statistics → Conversations → TCP (no filter)

![TCP Conversations 3076](Bildschirmfoto%202026-04-11%20um%2014.49.03.png)

3076 total TCP conversations:
- Many ports, 1–3 packets each → nmap port scan (T1595)
- Port 22, many short connections → Hydra brute force (T1110)
- Port 22, one long large-data connection → successful login (T1078)

---

## EV-17 — I/O Graph — Full Attack Timeline

**Menu:** Statistics → I/O Graph

![I/O Graph attack timeline](Bildschirmfoto%202026-04-11%20um%2015.15.53.png)

```
T+60s   → Peak 1 (~2000 pkt/s) → nmap -sS -A -T4   SYN Scan
T+100s  → Peak 2 (~2000 pkt/s) → nmap -O            OS Fingerprinting
T+120s  → Peak 3 (~2000 pkt/s) → nmap -sV           Service Detection
T+200s  → Peak 4 ( ~350 pkt/s) → Hydra              SSH Brute Force
T+390s  → Red TCP Error zone    → fail2ban            Kali IP blocked
```

Entire attack timeline reconstructed from one PCAP file. No SIEM. No logs. Packets only.

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
