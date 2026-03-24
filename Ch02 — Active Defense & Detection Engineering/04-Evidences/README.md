# 04 — Evidences

## Phase 1 — Wazuh Active Response

### iptables DROP — Attacker IP Blocked

Wazuh Active Response triggered `firewall-drop` command upon SSH brute force detection. Kali's IP (172.20.10.3) was automatically added to iptables DROP chain.

![iptables DROP](Bildschirmfoto%202026-03-24%20um%2009.42.22.png)

---

### Wazuh Dashboard — Active Response Events

Dashboard query: `rule.id:651 or rule.id:652`

| Rule | Description | Timestamp |
|:-----|:------------|:---------|
| 651 | Host Blocked by firewall-drop Active Response | 09:41:44 |
| 652 | Host Unblocked by firewall-drop Active Response | 09:46:45 |
| 651 | Host Blocked by firewall-drop Active Response | 09:53:17 |

The 300-second timeout was confirmed: blocked → unblocked → re-blocked on second attack.

![Dashboard Active Response](Bildschirmfoto%202026-03-24%20um%2009.54.38.png)

---

## Phase 2 — Custom Suricata Rule

### eve.json — AEGIS SSH Brute Force Attempt

Custom rule `sid:9000001` triggered and logged to eve.json. Alert message: **"AEGIS SSH Brute Force Attempt"**

![eve.json AEGIS Alert](Bildschirmfoto%202026-03-24%20um%2011.40.49.png)

---

## Phase 3 — fail2ban

### fail2ban — Banned IP List

fail2ban detected 5 failed SSH attempts within 60 seconds and automatically banned 172.20.10.3.

![fail2ban Banned IP](Bildschirmfoto%202026-03-24%20um%2012.31.41.png)

---

### Wazuh Dashboard — fail2ban Alert

Dashboard query: `rule.id:100100`

Custom rule 100100 triggered — **"Fail2ban: Host banned"** visible in Dashboard with full log details.

![Dashboard fail2ban Alert](Bildschirmfoto%202026-03-24%20um%2014.38.10.png)

---

## Phase 4 — MITRE ATT&CK Dashboard

### MITRE Dashboard — Top Tactics & Techniques

![MITRE Dashboard](Bildschirmfoto%202026-03-24%20um%2015.00.40.png)

---

### MITRE Framework — Tactics & Technique Count

| Tactic | Alert Count |
|:-------|:-----------|
| Defense Evasion | 288 |
| Privilege Escalation | 277 |
| Credential Access | 168 |
| Persistence | 141 |
| Initial Access | 141 |
| Lateral Movement | 109 |

![MITRE Framework](Bildschirmfoto%202026-03-24%20um%2015.06.17.png)

---

### MITRE Events — T1110 Fail2ban Visible

fail2ban ban events confirmed in MITRE Events tab — `T1110 · Credential Access · Fail2ban: Host banned · rule.id:100100`

![MITRE Events](Bildschirmfoto%202026-03-24%20um%2015.16.58.png)
