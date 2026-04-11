# 04 — Evidences

## EV-01 — nmap SYN Scan im PCAP

**Filter:** `tcp.flags.syn == 1 && tcp.flags.ack == 0`  
**Ergebnis:** 3074 Pakete (34.4%)

![nmap SYN Scan Pakete](Bildschirmfoto%202026-04-11%20um%2013.19.56.png)

**Was wir sehen:**
- Source: `192.168.178.129` (Kali) → alle Pakete kommen vom Angreifer
- `Win=1024` → nmap Fingerabdruck! Normaler Traffic hat `Win=64240`
- SYN Pakete an hunderte verschiedene Ports in Millisekunden → Port Scan

---

## EV-02 — Win=1024 vs Win=64240

![Win Size Vergleich](Bildschirmfoto%202026-04-11%20um%2013.23.06.png)

**Zwei verschiedene Muster:**

| Win Size | Tool | Bedeutung |
|:---------|:-----|:----------|
| `Win=1024` | nmap | Halb-offene Verbindung, kein echter Transfer |
| `Win=64240` | Hydra | Echte Verbindung, Daten werden übertragen |

---

## EV-03 — SSH Traffic auf Port 22

**Filter:** `tcp.port == 22 && tcp.flags.syn == 1`  
**Ergebnis:** 134 Pakete (1.5%)

![SSH SYN Pakete](Bildschirmfoto%202026-04-11%20um%2013.29.15.png)

**Was wir sehen:**
- `Win=1024` Pakete → nmap scannt Port 22
- `Win=64240` Pakete → Hydra baut echte SSH Verbindungen auf
- Rote Zeilen `[FIN, SYN, PSH, URG]` → nmap OS Fingerprinting Probe

---

## EV-04 — Gesamter SSH Traffic

**Filter:** `tcp.port == 22`  
**Ergebnis:** 2191 Pakete (24.5%)

![Gesamter SSH Traffic](Bildschirmfoto%202026-04-11%20um%2013.36.55.png)

**Wichtig:** Hier sehen wir zwei verschiedene Quellen:
- `192.168.178.122` (MacBook) → legitime SSH Admin Session
- `192.168.178.129` (Kali) → Angriff

tcpdump nimmt **alles** auf — auch eigene Verbindungen!

---

## EV-05 — MacBook SSH Session entdeckt

![MacBook SSH Session](Bildschirmfoto%202026-04-11%20um%2013.38.08.png)

`192.168.178.122` ist das MacBook — keine Bedrohung. Als SOC Analyst muss man Kontext kennen um legitimen Traffic von Angriffen zu unterscheiden.

---

## EV-06 — nmap SSH Banner (Smoking Gun)

**Filter:** `tcp.port == 22`  
**Zeilen 2394-2395:**

![nmap SSH Banner](Bildschirmfoto%202026-04-11%20um%2013.55.14.png)

```
Client: Protocol (SSH-1.5-NmapNSE-1.0)
Client: Protocol (SSH-1.5-Nmap-SSH1-Hostkey)
Server: Protocol (SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.15)
```

nmap identifiziert sich im Klartext! Ein SOC Analyst sieht `NmapNSE` und weiß sofort: das ist kein echter SSH Client.

---

## EV-07 — Angreifer IP isoliert

**Filter:** `ip.addr == 192.168.178.129`  
**Ergebnis:** 7970 Pakete (89.2%)

![Angreifer Traffic](Bildschirmfoto%202026-04-11%20um%2013.56.01.png)

89% des gesamten Traffics kam vom Angreifer — eindeutig kein normales Verhalten.

---

## EV-08 — OS Fingerprinting Pakete

![OS Fingerprinting](Bildschirmfoto%202026-04-11%20um%2013.56.48.png)

**Rote Zeilen:**
```
[RST] Win=0   → aegis-sentinel blockt die Verbindung (fail2ban!)
```

**ICMP Zeilen:**
```
Echo request ttl=40  → nmap pingt zuerst ob Host erreichbar
Echo reply   ttl=64  → Host antwortet
```

**UDP Zeile:**
```
ICMP Destination unreachable → Port existiert nicht
```

---

## EV-09 — nmap Stream isoliert

**Filter:** `tcp.stream eq 1017`

![nmap Stream](Bildschirmfoto%202026-04-11%20um%2014.08.48.png)

Komplette nmap SSH Verbindung in einer Ansicht:
```
SYN → Verbindung anfragen
SYN-ACK → Port offen!
Server Banner → OpenSSH 9.6p1 Ubuntu
Client Banner → SSH-2.0-NmapNSE ← Angreifer identifiziert!
FIN → sofort wieder getrennt
```

---

## EV-10 — Follow Stream — Cipher Suites

![Follow Stream](Bildschirmfoto%202026-04-11%20um%2014.11.54.png)

nmap liest alle unterstützten Cipher Suites des Servers — vollständige Aufklärung der Verschlüsselungskonfiguration.

---

## EV-11 — Hydra Brute Force Muster

**Filter:** `tcp.port == 22 && tcp.len > 100`  
**Ergebnis:** 123 Pakete (1.4%)

![Hydra Key Exchange Muster](Bildschirmfoto%202026-04-11%20um%2014.33.01.png)

**Oben (wiederholend):**
```
Key Exchange Init (1042 bytes)  ← ein Passwortversuch
ECDH Reply (558 bytes)          ← Verbindung fehlgeschlagen
Key Exchange Init (1042 bytes)  ← nächster Versuch
...
```

**Unten (einmalig):**
```
Encrypted packet (len=628)  ← großes Paket!
Encrypted packet (len=528)  ← aktive Session!
```

→ Das Passwort wurde gefunden!

---

## EV-12 — Erfolgreicher Login bewiesen

![Erfolgreicher Login](Bildschirmfoto%202026-04-11%20um%2014.40.29.png)

Nach hunderten kurzen Verbindungen plötzlich eine **lange verschlüsselte Session** — Hydra ist eingeloggt. Beweisbar nur aus dem PCAP, ohne Hydra Terminal Output.

---

## EV-13 — Conversations — 3076 TCP

**Menü:** Statistics → Conversations → TCP

![TCP Conversations](Bildschirmfoto%202026-04-11%20um%2014.49.03.png)

**3076 TCP Verbindungen** — davon:
- Viele Ports, je 1 Verbindung → nmap Port Scan
- Port 22, viele Verbindungen → Hydra Brute Force
- Port 22, große Datenpakete → erfolgreicher Login

---

## EV-14 — I/O Graph — Angriffs-Timeline

**Menü:** Statistics → I/O Graph

![I/O Graph Timeline](Bildschirmfoto%202026-04-11%20um%2015.15.53.png)

```
T+60s   → Peak 1 (2000 pkt/s) → nmap -sS    SYN Scan
T+100s  → Peak 2 (2000 pkt/s) → nmap -O     OS Fingerprinting
T+120s  → Peak 3 (2000 pkt/s) → nmap -sV    Service Detection
T+200s  → Peak 4 ( 350 pkt/s) → Hydra       Brute Force
T+390s  → Rote Zone (TCP Errors) → fail2ban  Kali geblockt!
```

Der gesamte Angriff rekonstruiert — **ohne SIEM, ohne Logs, nur aus Paketen.**

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
