# Ch.03 — Network Forensics & PCAP Analysis

**homelab_AEGIS** · SOC from Scratch · April 11, 2026

---

## Was haben wir hier gemacht?

In Ch.01 haben wir Angriffe **gesehen**. In Ch.02 haben wir sie **gestoppt**. In Ch.03 stellen wir eine andere Frage:

> *Wenn du nur die Netzwerkdaten hättest — kein SIEM, keine Logs — könntest du den Angriff trotzdem rekonstruieren?*

Die Antwort: **Ja.**

Wir haben den Angriff aus Ch.01/02 wiederholt, dabei den gesamten Traffic mit `tcpdump` aufgenommen und anschließend mit Wireshark forensisch analysiert. Das Ergebnis: Angreifer-IP, verwendete Tools, Angriffsmuster und sogar der erfolgreiche Login — alles aus dem PCAP rekonstruiert.

---

## Lab Umgebung

| Maschine | Rolle | OS | IP |
|:---------|:------|:---|:---|
| MacBook Pro | Analyse (Wireshark) | macOS | 192.168.178.122 |
| Kali Laptop | Angreifer | Kali Linux | 192.168.178.129 |
| aegis-sentinel VM | Capture Node | Ubuntu 24.04.4 | 192.168.178.126 |

---

## Ablauf

```
aegis-sentinel: tcpdump startet → nimmt alles auf
        ↓
Kali: nmap -sS -A -T4     (SYN Scan)
Kali: nmap -O             (OS Fingerprinting)
Kali: nmap -sV            (Service Detection)
Kali: hydra SSH           (Brute Force → Passwort: 1111 ✅)
        ↓
aegis-sentinel: tcpdump stoppt → 8936 Pakete gespeichert
        ↓
MacBook: Wireshark → PCAP analysieren
```

---

## Was wir im PCAP beweisen konnten

| Frage | Antwort | Beweis |
|:------|:--------|:-------|
| Wer hat angegriffen? | 192.168.178.129 | 89.2% aller Pakete |
| Welches Tool? | nmap | Win=1024 + SSH-2.0-NmapNSE Banner |
| Welcher Service? | SSH Port 22 | 2191 Pakete auf Port 22 |
| Wie viele Versuche? | 3076 TCP Verbindungen | Statistics → Conversations |
| War der Angriff erfolgreich? | Ja | Große verschlüsselte Pakete nach Key Exchange |
| Wann geblockt? | T+390s | RST Pakete — fail2ban aus Ch.02 |

---

## MITRE ATT&CK

| Technik | ID | Tool | Beweis im PCAP |
|:--------|:---|:-----|:---------------|
| Active Scanning | T1595 | nmap | SYN Flood, Win=1024 |
| Active Scanning | T1595 | nmap -O | FIN+SYN+PSH+URG Flags |
| Active Scanning | T1595 | nmap -sV | SSH-2.0-NmapNSE Banner |
| Brute Force | T1110.001 | Hydra | Key Exchange Muster |
| Valid Accounts | T1078 | Hydra | Aktive verschlüsselte Session |

---

## Chapter Struktur

| Ordner | Inhalt |
|:-------|:-------|
| [01-Architecture](01-Architecture/) | Lab Umgebung, Topologie, Capture Setup |
| [02-Assets](02-Assets/) | Tools, PCAP Info, Wireshark Filter |
| [03-Attack_Scenario](03-Attack_Scenario/) | Angriffsbefehle + Screenshots |
| [04-Evidences](04-Evidences/) | Wireshark Screenshots + Analyse |
| [05-LESSONS_LEARNED](05-LESSONS_LEARNED/) | Fehler, Lösungen, Erkenntnisse |

---

*homelab_AEGIS · github.com/cyb-ersin · Built from scratch, documented from zero.*
