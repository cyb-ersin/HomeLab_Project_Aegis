# 01 — Architecture

## Network Topology

![Lab Topology](topology.svg)

---

## Lab Environment

| Machine | Role | OS | IP |
|:--------|:-----|:---|:---|
| MacBook Pro | Wireshark Analysis | macOS | 192.168.178.122 |
| Kali Laptop | Attacker | Kali Linux | 192.168.178.129 |
| aegis-sentinel VM | Capture Node | Ubuntu 24.04.4 LTS | 192.168.178.126 |

**Network:** Wireless Access Point · 192.168.178.0/24  
**Note:** IP range changed from Ch.01/02 (172.20.10.0/28) — new network session. Wazuh not required for this chapter.

---

## Capture Setup

`tcpdump` was started on aegis-sentinel **before** the attack began:

```bash
sudo tcpdump -i enp0s3 -w /tmp/ch03_full.pcap
```

| Flag | Meaning |
|:-----|:--------|
| `-i enp0s3` | Listen on this interface |
| `-w /tmp/ch03_full.pcap` | Write packets to file for offline analysis |
| no filter | Capture everything — filter later in Wireshark |

**Result:**
```
tcpdump: listening on enp0s3, link-type EN10MB
8936 packets captured
8945 packets received by filter
0 packets dropped by kernel
```

---

## PCAP Transfer

After the attack, the PCAP file was transferred from aegis-sentinel to MacBook via `scp`:

```bash
scp aegis-siem@192.168.178.126:/tmp/ch03_full.pcap ~/Desktop/ch03_full.pcap
```

Wireshark opened the file locally on MacBook for offline forensic analysis.

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
