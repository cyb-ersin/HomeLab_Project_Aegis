# 02 — Assets

## Tool Versions

| Tool | Version | Host | Role |
|:-----|:--------|:-----|:-----|
| tcpdump | 4.99.x | aegis-sentinel | Live packet capture |
| Wireshark | 4.x | MacBook Pro | Offline PCAP analysis |
| nmap | 7.98 | Kali Laptop | Reconnaissance + OS/Service detection |
| Hydra | 9.6 | Kali Laptop | SSH brute force |

---

## PCAP File

| Property | Value |
|:---------|:------|
| Filename | ch03_full.pcap |
| Location (capture) | /tmp/ch03_full.pcap (aegis-sentinel) |
| Location (analysis) | ~/Desktop/ch03_full.pcap (MacBook) |
| Total packets | 8936 |
| File size | ~1.9 MB |
| Capture duration | ~650 seconds |
| Interface | enp0s3 |

---

## Wordlist Used

| File | Path | Size |
|:-----|:-----|:-----|
| rockyou.txt | /usr/share/wordlists/rockyou.txt | 14.3M passwords |
| rockyou_9800.txt | /tmp/rockyou_9800.txt | Custom — starting from line 9800 |

**Password position:**
```bash
grep -n "^111$" /usr/share/wordlists/rockyou.txt
# Output: 9845
```

Password `111` is at line 9845 in rockyou.txt — custom list started at 9800 to reach it quickly.

---

## tcpdump Command

```bash
sudo tcpdump -i enp0s3 -w /tmp/ch03_full.pcap
```

| Flag | Meaning |
|:-----|:--------|
| `-i enp0s3` | Listen on interface enp0s3 |
| `-w /tmp/ch03_full.pcap` | Write raw packets to file (offline analysis) |
| no filter | Capture everything — filter in Wireshark |

---

## Wireshark Filters Used

| Filter | Purpose | Result |
|:-------|:--------|:-------|
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | nmap SYN scan packets | 3074 pkts (34.4%) |
| `tcp.port == 22 && tcp.flags.syn == 1` | SSH SYN packets only | 134 pkts (1.5%) |
| `tcp.port == 22` | All SSH traffic | 2191 pkts (24.5%) |
| `ip.addr == 192.168.178.129` | All attacker traffic | 7970 pkts (89.2%) |
| `tcp.port == 22 && tcp.len > 100` | Large SSH packets (Hydra pattern) | 123 pkts (1.4%) |
| `tcp.stream eq 1017` | nmap SSH banner stream isolated | 14 pkts |

---

## Ch.02 Defense — Still Active

fail2ban and Wazuh Active Response rules from Ch.02 were still running on aegis-sentinel.
This caused Kali's IP to be blocked after 5 failed SSH attempts — visible as RST packets in PCAP.

**Reset commands used:**
```bash
sudo iptables -F
sudo fail2ban-client unban --all
```

**Also encountered:** ufw on Kali Laptop had `Chain INPUT (policy DROP)` — blocked scp from MacBook.

```bash
sudo ufw disable   # on Kali
```

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
