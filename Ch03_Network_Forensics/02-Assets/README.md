# 02 — Assets

## Tools

| Tool | Version | Maschine | Rolle |
|:-----|:--------|:---------|:------|
| tcpdump | 4.99.x | aegis-sentinel | Pakete aufnehmen |
| Wireshark | 4.x | MacBook Pro | PCAP analysieren |
| nmap | 7.98 | Kali Laptop | Reconnaissance |
| Hydra | 9.6 | Kali Laptop | SSH Brute Force |

---

## PCAP Datei

| Eigenschaft | Wert |
|:------------|:-----|
| Dateiname | ch03_full.pcap |
| Größe | ~1.9 MB |
| Pakete gesamt | 8936 |
| Dauer | ~650 Sekunden |
| Interface | enp0s3 |

---

## Passwortliste

Das Passwort `111` steht in rockyou.txt auf **Zeile 9845**:

```bash
grep -n "^111$" /usr/share/wordlists/rockyou.txt
# Output: 9845
```

Wir haben eine kleinere Liste ab Zeile 9800 erstellt damit Hydra schnell findet:

```bash
tail -n +9800 /usr/share/wordlists/rockyou.txt > /tmp/rockyou_9800.txt
```

---

## Wireshark Filter

| Filter | Was er zeigt | Ergebnis |
|:-------|:-------------|:---------|
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | nmap SYN Scan | 3074 Pakete |
| `tcp.port == 22 && tcp.flags.syn == 1` | SSH SYN Pakete | 134 Pakete |
| `tcp.port == 22` | Gesamter SSH Traffic | 2191 Pakete |
| `ip.addr == 192.168.178.129` | Angreifer Traffic | 7970 Pakete (89.2%) |
| `tcp.port == 22 && tcp.len > 100` | Große SSH Pakete | 123 Pakete |
| `tcp.stream eq 1017` | nmap SSH Stream | 14 Pakete |

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
