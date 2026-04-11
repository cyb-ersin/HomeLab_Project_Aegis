# 02 — Assets

## Tools

| Tool | Version | Machine | Role |
|:-----|:--------|:--------|:-----|
| tcpdump | 4.99.x | aegis-sentinel | Live packet capture |
| Wireshark | 4.x | MacBook Pro | Offline PCAP analysis |
| nmap | 7.98 | Kali Laptop | Reconnaissance |
| Hydra | 9.6 | Kali Laptop | SSH brute force |

---

## PCAP File

| Property | Value |
|:---------|:------|
| Filename | ch03_full.pcap |
| Size | ~1.9 MB |
| Total packets | 8936 |
| Duration | ~650 seconds |
| Interface | enp0s3 |

---

## Password List

The password `111` is at **line 9845** in rockyou.txt:

```bash
grep -n "^111$" /usr/share/wordlists/rockyou.txt
# Output: 9845
```

A smaller list was created starting from line 9800 so Hydra finds it quickly:

```bash
tail -n +9800 /usr/share/wordlists/rockyou.txt > /tmp/rockyou_9800.txt
```

---

## Wireshark Filters Used

| Filter | What it shows | Result |
|:-------|:-------------|:-------|
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | nmap SYN scan packets | 3074 pkts (34.4%) |
| `tcp.port == 22 && tcp.flags.syn == 1` | SSH SYN packets only | 134 pkts (1.5%) |
| `tcp.port == 22` | All SSH traffic | 2191 pkts (24.5%) |
| `ip.addr == 192.168.178.129` | All attacker traffic | 7970 pkts (89.2%) |
| `tcp.port == 22 && tcp.len > 100` | Large SSH packets | 123 pkts (1.4%) |
| `tcp.stream eq 1017` | nmap SSH stream isolated | 14 pkts |

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
