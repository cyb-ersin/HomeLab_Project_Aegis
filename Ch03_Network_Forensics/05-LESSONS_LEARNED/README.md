# 05 — Lessons Learned

## Key Findings

| # | Finding | Status |
|:--|:--------|:-------|
| 1 | Angriff komplett aus PCAP rekonstruiert — ohne SIEM | ✅ Bewiesen |
| 2 | nmap durch `Win=1024` eindeutig identifizierbar | ✅ Dokumentiert |
| 3 | nmap durch SSH Banner `SSH-2.0-NmapNSE` im Klartext sichtbar | ✅ Dokumentiert |
| 4 | Erfolgreicher Login durch große Paketgrößen beweisbar | ✅ Bestätigt |
| 5 | fail2ban aus Ch.02 war noch aktiv — hat Kali nach 5 Versuchen geblockt | ✅ Defense-in-Depth funktioniert |
| 6 | MacBook SSH Session im PCAP sichtbar — Analyst-Kontext notwendig | ⚠️ Erwartet |
| 7 | ufw auf Kali hatte `policy DROP` — scp vom MacBook blockiert | ✅ Gelöst |

---

## Fehler & Lösungen

### Fehler 1 — tcpdump Syntax Error

**Problem:**
```
tcpdump: can't parse filter expression: syntax error
```

**Ursache:** Copy-Paste hat unsichtbare Sonderzeichen eingefügt.

**Lösung:** Befehl manuell eintippen:
```bash
sudo tcpdump -i enp0s3 -w /tmp/ch03_full.pcap
```

---

### Fehler 2 — Hydra findet Passwort nicht

**Problem:** Hydra hat `111` (Zeile 9845) übersprungen obwohl es richtig war.

**Ursache:** fail2ban aus Ch.02 war noch aktiv. Nach 5 Versuchen wurde Kali geblockt — Hydra bekam `Connection refused` und interpretierte das als falsches Passwort.

**Lösung:**
```bash
sudo iptables -F
sudo fail2ban-client unban --all
tail -n +9800 /usr/share/wordlists/rockyou.txt > /tmp/rockyou_9800.txt
hydra -l aegis-siem -P /tmp/rockyou_9800.txt ssh://192.168.178.126
```

---

### Fehler 3 — scp vom MacBook funktioniert nicht

**Problem:**
```
ssh: connect to host 192.168.178.129 port 22: Operation timed out
```

**Ursache:** ufw auf Kali hatte `Chain INPUT (policy DROP)` — alle eingehenden Verbindungen geblockt.

**Lösung:**
```bash
sudo ufw disable   # auf Kali
```

---

## Was haben wir gelernt?

**Network Forensics funktioniert ohne SIEM.** Der vollständige Angriff — Reconnaissance, Brute Force und erfolgreicher Login — wurde nur aus Netzwerkpaketen rekonstruiert.

**nmap hinterlässt eindeutige Spuren.** `Win=1024` und `SSH-2.0-NmapNSE` sind unverwechselbare Fingerabdrücke. Ein SOC Analyst erkennt nmap sofort.

**Paketgröße verrät den Inhalt.** Auch in verschlüsseltem Traffic zeigt die Paketgröße das Verhalten — viele kleine Pakete = Brute Force, plötzlich große Pakete = aktive Session.

**Defense-in-Depth hat Zeitwirkung.** fail2ban Regeln aus Ch.02 wirkten in Ch.03 weiter — Sicherheitsmaßnahmen persistieren über Vorfälle hinaus.

**tcpdump nimmt alles auf.** Keine Filter beim Aufnehmen — erst in Wireshark filtern. Fehlende Pakete können nicht mehr rekonstruiert werden.

---

## Verbindung zu Ch.04

Ch.03 hat bewiesen dass Angriffstraffic forensisch sichtbar und identifizierbar ist.

Die offene Frage: **Können wir Exploitation in Echtzeit erkennen?**

Ch.04 — Exploitation & Detection:
- Metasploit Shell gegen aegis-sentinel
- Erkennt Suricata eine Reverse Shell?
- Kann Wazuh Exploit + Post-Exploitation korrelieren?

> Forensics zeigt was passiert ist. Detection verhindert dass es nochmal passiert.

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
