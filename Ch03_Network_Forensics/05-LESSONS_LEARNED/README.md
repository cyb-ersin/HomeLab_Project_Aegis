# 05 — Lessons Learned

## Key Findings

| # | Finding | Status |
|:--|:--------|:-------|
| 1 | Full attack reconstructed from PCAP — no SIEM required | ✅ Proven |
| 2 | nmap fingerprinted via `Win=1024` window size | ✅ Documented |
| 3 | nmap identified via SSH banner `SSH-2.0-NmapNSE` in cleartext | ✅ Documented |
| 4 | Successful login confirmed via encrypted packet size shift | ✅ Confirmed |
| 5 | fail2ban from Ch.02 still active — blocked Kali after 5 attempts | ✅ Defense-in-depth working |
| 6 | MacBook SSH session captured alongside attack traffic | ⚠️ Expected — context required |
| 7 | ufw on Kali had `policy DROP` — blocked scp from MacBook | ✅ Resolved |
| 8 | iptables rules from Ch.02 persisted across sessions | ✅ Documented |

---

## Errors & Solutions

### Error 1 — tcpdump Syntax Error

**Symptom:**
```
tcpdump: can't parse filter expression: syntax error
```

**Cause:** Copy-paste introduced invisible special characters.

**Solution:** Type the command manually:
```bash
sudo tcpdump -i enp0s3 -w /tmp/ch03_full.pcap
```

---

### Error 2 — Hydra Skipped Correct Password

**Symptom:** Hydra ran through the list but skipped `111` (line 9845).

**Cause:** fail2ban from Ch.02 was still active. After 5 failed attempts Kali's IP was blocked — Hydra received `Connection refused` for every subsequent attempt including the correct password, and treated it as a wrong credential.

**Solution:**
```bash
sudo iptables -F
sudo fail2ban-client unban --all
tail -n +9800 /usr/share/wordlists/rockyou.txt > /tmp/rockyou_9800.txt
hydra -l aegis-siem -P /tmp/rockyou_9800.txt ssh://192.168.178.126
```

---

### Error 3 — scp Connection Timeout from MacBook

**Symptom:**
```
ssh: connect to host 192.168.178.129 port 22: Operation timed out
```

**Cause:** ufw on Kali had `Chain INPUT (policy DROP)` — blocking all incoming connections.

**Solution:**
```bash
sudo ufw disable   # on Kali
```

---

## Key Takeaways

**Network forensics works without a SIEM.** The complete attack — reconnaissance, brute force, and successful login — was reconstructed from raw packet data alone. No Wazuh, no Suricata, no system logs.

**nmap leaves obvious fingerprints.** `Win=1024` and `SSH-2.0-NmapNSE` are unmistakable signatures. A SOC analyst seeing either can confirm nmap with certainty.

**Packet size reveals behavior.** Even in encrypted traffic, packet size and timing tells the story. Repeated small Key Exchange packets = brute force. Sudden large encrypted packets = active session.

**Defense-in-depth has temporal reach.** Ch.02 fail2ban rules actively impacted Ch.03 — security controls persist and layer across incidents over time.

**Capture everything, filter later.** No filter at capture time = complete evidence. Apply filters in Wireshark for analysis. Missing packets cannot be recovered.

**Context separates signal from noise.** The MacBook SSH session appeared alongside attack traffic. Without knowing the environment, it could be misidentified. Analyst context is essential.

---

## Connection to Ch.04

Ch.03 proved that attack traffic is forensically visible and identifiable from packets alone.

The open question: **can we detect exploitation in real time?**

**Ch.04 — Exploitation & Detection:**
- Metasploit shell against aegis-sentinel
- Does Suricata detect a reverse shell?
- Can Wazuh correlate exploit + post-exploitation activity?

> Forensics shows what happened. Detection prevents it from happening again.

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
