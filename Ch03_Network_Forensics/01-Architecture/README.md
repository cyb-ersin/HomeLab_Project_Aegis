# 01 — Architecture

## Lab Environment

| Machine | Role | OS | IP |
|:--------|:-----|:---|:---|
| MacBook Pro (physical) | Management + Wireshark Analysis | macOS | 192.168.178.122 |
| Kali Laptop (physical) | Attacker | Kali Linux | 192.168.178.129 |
| Kali VirtualBox → aegis-sentinel | Capture Node | Ubuntu 24.04.4 LTS | 192.168.178.126 |

**Network:** Wireless Access Point · 192.168.178.0/24 · all machines on same subnet

**Note:** IP range changed from Ch.01/02 (172.20.10.0/28) — new network environment. Wazuh Manager not required for this chapter.

---

## Network Diagram

```
Wireless Access Point (192.168.178.0/24)
        │
        ├── MacBook Pro (192.168.178.122)
        │       │
        │       └── Wireshark — offline PCAP analysis
        │
        ├── Kali Laptop (192.168.178.129)
        │       │
        │       ├── nmap -sS -A -T4   → Reconnaissance
        │       ├── nmap -O           → OS Fingerprinting
        │       ├── nmap -sV          → Service Detection
        │       └── hydra             → SSH Brute Force
        │
        └── aegis-sentinel VM (192.168.178.126)
                │
                ├── tcpdump → /tmp/ch03_full.pcap
                ├── fail2ban (still active from Ch.02)
                └── iptables (still active from Ch.02)
```

---

## Capture Setup

`tcpdump` was started on aegis-sentinel **before** the attack — capturing all traffic on `enp0s3`:

```bash
sudo tcpdump -i enp0s3 -w /tmp/ch03_full.pcap
```

**Result:**
```
tcpdump: listening on enp0s3, link-type EN10MB (Ethernet), snapshot length 262144 bytes
8936 packets captured
8945 packets received by filter
0 packets dropped by kernel
```

→ See [04-Evidences](../04-Evidences/) for tcpdump screenshot.

---

## PCAP Transfer

After attack completion, PCAP transferred from aegis-sentinel to MacBook via `scp`:

```bash
scp aegis-siem@192.168.178.126:/tmp/ch03_full.pcap ~/Desktop/ch03_full.pcap
```

Wireshark opened the file locally on MacBook for offline forensic analysis.

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
