# 05 — Lessons Learned

## Key Findings

| # | Finding | Severity | Status |
|:--|:--------|:---------|:-------|
| 1 | Full attack reconstructed from PCAP without SIEM or logs | — | ✅ Proven |
| 2 | nmap identified by Win=1024 window size fingerprint | — | ✅ Documented |
| 3 | nmap identified by SSH banner (SSH-2.0-NmapNSE) in cleartext | High | ✅ Documented |
| 4 | Successful Hydra login confirmed via large encrypted packet shift | High | ✅ Confirmed |
| 5 | fail2ban from Ch.02 still active — blocked Kali before all passwords tried | — | ✅ Defense-in-Depth working |
| 6 | MacBook admin SSH session captured alongside attack — requires analyst context | Low | ⚠️ Expected |
| 7 | ufw on Kali had policy DROP — blocked scp from MacBook | Low | ✅ Resolved |
| 8 | iptables rules from Ch.02 persisted across sessions | Low | ✅ Documented |

---

## Errors Encountered & Solutions

### Error 1 — tcpdump Syntax Error

**Symptom:**
```
tcpdump: can't parse filter expression: syntax error
```

**Root Cause:** Copy-paste introduced invisible special characters into the command.

**Solution:** Typed the command manually character by character.

```bash
sudo tcpdump -i enp0s3 -w /tmp/ch03_full.pcap
```

**Lesson:** Always type tcpdump commands manually — copy-paste from formatted sources can insert hidden characters.

---

### Error 2 — Hydra Not Finding Password

**Symptom:** Hydra ran through rockyou.txt but skipped password `111` (line 9845).

**Root Cause:** fail2ban from Ch.02 was still active. After 5 failed attempts, Kali's IP was blocked by iptables. Hydra received `Connection refused` for every subsequent attempt — including the correct password — and interpreted it as wrong credentials.

**Diagnosis:**
```bash
sudo iptables -L -n     # showed REJECT rules still active
sudo fail2ban-client status sshd   # showed Kali IP in ban list
```

**Solution:**
```bash
sudo iptables -F
sudo fail2ban-client unban --all
```

Then created a targeted password list starting near the correct password:
```bash
tail -n +9800 /usr/share/wordlists/rockyou.txt > /tmp/rockyou_9800.txt
hydra -l aegis-siem -P /tmp/rockyou_9800.txt ssh://192.168.178.126
```

**Lesson:** Always reset defensive tools between lab phases. Ch.02 defense actively interfered with Ch.03 attack — this is realistic behavior and demonstrates defense-in-depth working correctly.

---

### Error 3 — scp Connection Timeout from MacBook

**Symptom:**
```
ssh: connect to host 192.168.178.129 port 22: Operation timed out
```

**Root Cause 1:** SSH service not running on Kali.
```bash
sudo systemctl start ssh   # fixed service
```

**Root Cause 2 (main cause):** ufw on Kali Laptop had `Chain INPUT (policy DROP)` — blocking all incoming connections including SSH from MacBook.
```
Chain INPUT (policy DROP)
Chain FORWARD (policy DROP)
```

**Solution:**
```bash
sudo ufw disable   # on Kali
```

**Lesson:** ufw policy DROP blocks all incoming traffic by default — including your own management connections. Always check firewall state before troubleshooting connectivity issues.

---

### Error 4 — Wrong IP in Capture

**Symptom:** First PCAP captured with old IP (172.20.10.6) — network had changed.

**Solution:** Verified current IP with `ip addr show` and restarted capture with correct interface.

**Lesson:** Always verify network configuration at session start. DHCP can change IPs between lab sessions.

---

## Forensics — What We Proved

| Question | Answer | Evidence |
|:---------|:-------|:---------|
| Who attacked? | 192.168.178.129 (Kali) | ip.addr filter — 89.2% of all traffic |
| What tool was used for scanning? | nmap | Win=1024 + SSH-2.0-NmapNSE banner |
| What OS fingerprinting technique? | TCP probe flags | FIN+SYN+PSH+URG combined |
| What service was targeted? | SSH Port 22 | tcp.port==22 — 2191 packets |
| How many attempts? | 3076 TCP conversations | Statistics → Conversations |
| Was the attack successful? | Yes | Large encrypted packets post-Key Exchange |
| When was attacker blocked? | T+390s | RST packets — red zone in I/O Graph |
| Was Ch.02 defense still active? | Yes | fail2ban RST + blocked login attempts |

---

## Mitigations

| Finding | Mitigation |
|:--------|:-----------|
| nmap fingerprint (Win=1024) | Deploy IDS rule to alert on Win=1024 from external IPs |
| nmap SSH banner (NmapNSE) | Monitor SSH banner strings — alert on non-standard clients |
| SSH brute force visible in PCAP | Rate limit SSH connections at network level, not just host |
| Successful login visible via packet size | Correlate authentication logs with PCAP for confirmation |
| Ch.02 rules persisted | Document cleanup procedures — run at end of each chapter |

---

## Key Takeaways

**Network forensics works without SIEM.** The full attack — reconnaissance, brute force, and successful login — was reconstructed entirely from raw packet data. No Wazuh, no Suricata alerts, no system logs required.

**nmap leaves obvious fingerprints.** `Win=1024` and `SSH-2.0-NmapNSE` are distinctive signatures that immediately identify the tool. A SOC analyst seeing either of these can confirm nmap with certainty.

**Packet size tells the story.** Even in encrypted traffic, the size and timing of packets reveals behavior. Repeated small Key Exchange packets = brute force. Sudden large encrypted packets = successful session.

**Context separates noise from signal.** The MacBook SSH session appeared alongside attack traffic. Without knowing the environment, this could be misidentified. Analyst context is essential.

**Defense-in-depth has temporal reach.** Ch.02 fail2ban rules actively impacted Ch.03 — demonstrating that security controls persist and layer across time, not just within a single incident.

**tcpdump captures everything.** No filter = complete evidence. Apply filters in Wireshark for analysis, not at capture time. Missing packets cannot be recovered.

---

## Connection to Next Chapter

Ch.03 proved that attack traffic is forensically visible and identifiable.

The gap that remains: **can we detect exploitation in real time?**

Ch.04 — Exploitation & Detection will answer this:
- Metasploit shell against aegis-sentinel
- Does Suricata detect a reverse shell connection?
- Can Wazuh correlate exploit + post-exploitation activity?

> Forensics shows what happened. Detection prevents it from happening again.

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.03 — Network Forensics & PCAP Analysis*
