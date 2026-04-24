# Ch02_Active_Defense_Detection_Engineering

**homelab_AEGIS** · SOC from Scratch · March 24, 2026

---

## Overview

This chapter documents the transition from passive detection to active defense on the aegis-sentinel node. Building on the Suricata + Wazuh pipeline established in Ch.01, three independent defense layers were deployed and validated: Wazuh Active Response, a custom Suricata detection rule, and a fail2ban integration pipeline. All layers were tested against live SSH brute force attacks and correlated with MITRE ATT&CK techniques via the Wazuh Dashboard.

**The core finding from Ch.01 that motivated this chapter:**

> *"IDS in passive mode — attacks detected but not blocked."*

By the end of Ch.02, that finding is closed.

---

## Lab Environment

| Machine | Role | OS | IP |
|:--------|:-----|:---|:---|
| MacBook Pro (physical) | Management + Wazuh Dashboard | macOS | 172.20.10.4 |
| MacBook VirtualBox → Wazuh OVA | SIEM Server | Ubuntu (OVA) | 172.20.10.9 |
| Kali Laptop (physical) | Attacker | Kali Linux | 172.20.10.3 |
| Kali VirtualBox → aegis-sentinel | Sensor + Defense Node | Ubuntu 24.04.4 LTS | 172.20.10.6 |

**Network:** Wireless Access Point · 172.20.10.0/28 · all machines on same subnet

---

## Architecture

```
Kali Laptop (physical) — Attacker (172.20.10.3)
        │
        │  hydra -l aegis-siem -P rockyou.txt ssh://172.20.10.6
        ▼
aegis-sentinel VM (172.20.10.6)
        │
        ├── Suricata IDS ──► eve.json ──► custom rule sid:9000001
        │
        ├── Wazuh Agent ──► Wazuh Manager (172.20.10.9)
        │       │                   │
        │       │            Active Response
        │       │            firewall-drop ──► iptables DROP 172.20.10.3
        │
        └── fail2ban ──► syslog ──► Wazuh Agent ──► Manager
                                                rule 100100 (T1110)
```

---

## Defense Stack

| Component | Version | Role |
|:----------|:--------|:-----|
| Wazuh Active Response | 4.14.4 | Auto-block via firewall-drop on rule 5763 |
| Suricata Custom Rule | sid:9000001 | Lab-specific SSH brute force signature |
| fail2ban | 1.0.x | Independent IP ban layer via iptables |
| iptables | 1.8.10 | Firewall backend for both Active Response and fail2ban |

---

## Chapter Structure

| Folder | Content |
|:-------|:--------|
| [01-Architecture](01-Architecture/) | Lab environment, defense stack, network diagram |
| [02-Assets](02-Assets/) | Tool versions, key configurations, ossec.conf blocks |
| [03-Attack-Scenario](03-Attack_Scenario/) | Attack commands, flag explanations, MITRE mapping |
| [04-Evidences](04-Evidences/) | iptables screenshots, Dashboard alerts, MITRE events |
| [05-LESSONS_LEARNED](05-LESSONS_LEARNED/) | Findings, errors encountered, mitigations, key takeaways |

---

## Phase 1 — Wazuh Active Response

Wazuh's built-in `firewall-drop` command was configured to automatically block the attacking IP upon SSH brute force detection (rule 5763).

**Configuration added to Wazuh Manager (`/var/ossec/etc/ossec.conf`):**

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5763</rules_id>
  <timeout>300</timeout>
</active-response>
```

**Validation — iptables after attack:**
```
Chain INPUT (policy ACCEPT)
target    prot opt source         destination
DROP      0    --   172.20.10.3   0.0.0.0/0
```

**Wazuh Dashboard confirmation (rule.id:651 or rule.id:652):**

| Rule | Description | Timestamp |
|:-----|:-----------|:---------|
| 651 | Host Blocked by firewall-drop Active Response | 09:41:44 |
| 652 | Host Unblocked by firewall-drop Active Response | 09:46:45 |
| 651 | Host Blocked by firewall-drop Active Response | 09:53:17 |

The 300-second timeout was confirmed: block at 09:41, unban at 09:46, re-block on second attack at 09:53.

→ See [04-Evidences](04-Evidences/) for iptables and Dashboard screenshots.

---

## Phase 2 — Custom Suricata Rule

A lab-specific Suricata detection rule was written to identify SSH brute force attempts targeting aegis-sentinel. This rule complements the ET Open ruleset with a signature tuned to the exact lab topology.

**Rule (`/var/lib/suricata/rules/local.rules`):**

```
alert tcp any any -> 172.20.10.6 22 (msg:"AEGIS SSH Brute Force Attempt"; flow:to_server; threshold:type threshold, track by_src, count 5, seconds 60; sid:9000001; rev:1;)
```

**Rule anatomy:**

| Parameter | Value | Description |
|:----------|:------|:------------|
| `alert tcp` | action + protocol | Monitor TCP traffic |
| `any any -> 172.20.10.6 22` | source → destination | Any source to sentinel SSH port |
| `threshold count 5, seconds 60` | trigger condition | 5 connections within 60 seconds |
| `sid:9000001` | signature ID | 9000000+ range reserved for custom rules |

**Key lesson — Suricata rule path:**

Suricata reads rules from `default-rule-path` defined in `suricata.yaml`, which points to `/var/lib/suricata/rules/`. Writing a rule to `/etc/suricata/rules/` without copying it to the correct path results in silent failure — the rule loads without error but never triggers. Always verify with:

```bash
sudo tail -20 /var/log/suricata/suricata.log | grep "rules successfully loaded"
```

**Detection confirmed in eve.json:**
```json
{"signature_id": 9000001, "signature": "AEGIS SSH Brute Force Attempt",
 "src_ip": "172.20.10.3", "dest_ip": "172.20.10.6", "dest_port": 22}
```

→ See [04-Evidences](04-Evidences/) for eve.json and Wazuh alert screenshots.

---

## Phase 3 — fail2ban Integration

fail2ban was deployed as a second, independent blocking layer. Unlike Wazuh Active Response which depends on the SIEM pipeline, fail2ban operates directly on auth logs — if the Wazuh stack fails, fail2ban continues protecting the host.

### fail2ban → Wazuh Pipeline Problem

fail2ban's default log target (`/var/log/fail2ban.log`) writes in a non-standard format:

```
2026-03-24 10:06:00,595 fail2ban.actions [15817]: NOTICE [sshd] Ban 172.20.10.3
```

Wazuh's pre-decoder expects:
```
timestamp  hostname  program[pid]:  message
```

The hostname field is missing — Wazuh cannot parse the log and no alerts are generated.

**Solution:** Redirect fail2ban output to syslog.

```
# /etc/fail2ban/fail2ban.conf
logtarget = SYSLOG
```

Syslog writes in the correct format:
```
2026-03-24T11:21:35+00:00 aegis-sentinel fail2ban.actions[31777]: NOTICE [sshd] Ban 172.20.10.3
```

Now the pre-decoder correctly identifies `program_name: fail2ban.actions`.

### Wazuh Agent Configuration

```xml
<!-- /var/ossec/etc/ossec.conf on aegis-sentinel -->
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/syslog</location>
</localfile>
```

### Custom Rule on Wazuh Manager

```xml
<!-- /var/ossec/etc/rules/local_rules.xml on Wazuh OVA -->
<group name="fail2ban,">
  <rule id="100100" level="6">
    <program_name>fail2ban.actions</program_name>
    <match>Ban</match>
    <description>Fail2ban: Host banned</description>
    <group>authentication_failed,</group>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>

  <rule id="100101" level="3">
    <program_name>fail2ban.actions</program_name>
    <match>Unban</match>
    <description>Fail2ban: Host unbanned</description>
  </rule>
</group>
```

**Important:** Custom rules must be defined on the **Manager**, not the Agent. The Agent collects and forwards logs — all rule matching happens on the Manager side.

**Dashboard confirmation (rule.id:100100):**
```
Fail2ban: Host banned · level 6 · T1110 · Credential Access
```

→ See [04-Evidences](04-Evidences/) for fail2ban status and Dashboard screenshots.

---

## Phase 4 — MITRE ATT&CK Dashboard Analysis

### Top Tactics Observed

| Tactic | Alert Count | Source |
|:-------|:-----------|:-------|
| Defense Evasion | 288 | sudo usage post-access |
| Privilege Escalation | 277 | sudo to ROOT |
| Credential Access | 168 | Hydra brute force |
| Persistence | 141 | Post-login activity |
| Initial Access | 141 | Login with cracked password |
| Lateral Movement | 109 | SSH remote access |

### Techniques Triggered

| Technique | ID | Alert Count | Tool |
|:----------|:---|:-----------|:-----|
| Password Guessing | T1110.001 | 156 | Hydra + rockyou.txt |
| Valid Accounts | T1078 | 141 | SSH with stolen credentials |
| Sudo and Sudo Caching | T1548.003 | 136 | sudo commands post-login |
| SSH | T1021.004 | 108 | Hydra SSH protocol |
| Brute Force | T1110 | 12 | fail2ban detection |
| Remote Services | T1021 | 1 | SSH remote service |

→ See [04-Evidences](04-Evidences/) for MITRE Framework and Events screenshots.

---

## MITRE ATT&CK Kill Chain

| Phase | Technique | ID | Tool | Detected | Blocked |
|:------|:----------|:---|:-----|:---------|:--------|
| Credential Access | Password Guessing | T1110.001 | Hydra + rockyou.txt | ✅ Wazuh rule 40111 | ✅ Active Response + fail2ban |
| Lateral Movement | Remote Services: SSH | T1021.004 | Hydra SSH | ✅ Suricata sid:9000001 | ✅ iptables DROP |
| Initial Access | Valid Accounts | T1078 | SSH with stolen creds | ✅ Wazuh rule 40112 | ✅ Active Response |
| Privilege Escalation | Sudo and Sudo Caching | T1548.003 | sudo commands | ✅ Wazuh rule 5402 | — |

---

## Defense Layers Summary

| Layer | Tool | Trigger | Action | MITRE |
|:------|:-----|:--------|:-------|:------|
| 1 | Wazuh Active Response | rule 5763 (level 10) | firewall-drop → iptables DROP | T1110 |
| 2 | Custom Suricata Rule | 5 SSH attempts / 60s | Alert in eve.json | T1110.001 |
| 3 | fail2ban | 5 failed logins / 60s | iptables DROP (independent) | T1110 |

---

## Key Findings

| # | Finding | Severity | Status |
|:--|:--------|:---------|:-------|
| 1 | Ch.01 gap closed — IDS now actively blocks attackers | — | ✅ Resolved |
| 2 | fail2ban log format incompatible with Wazuh pre-decoder by default | Medium | ✅ Resolved via SYSLOG redirect |
| 3 | Suricata rule path mismatch — `/etc/suricata/rules/` vs `/var/lib/suricata/rules/` | Medium | ✅ Resolved via cp |
| 4 | Custom Wazuh rules must be defined on Manager, not Agent | — | ✅ Documented |
| 5 | Two independent block layers provide defense-in-depth | — | ✅ Operational |
| 6 | sudo activity post-login generates highest alert volume (288) | Low | ⚠️ Monitor |

→ See [05-LESSONS_LEARNED](05-LESSONS_LEARNED/) for full analysis.

---

## Next — Ch.03

- PCAP forensics — capture and analyze attack traffic offline
- Wireshark deep-dive on brute force packet patterns
- Timeline reconstruction from network evidence

---

*homelab_AEGIS · github.com/cyb-ersin · Built from scratch, documented from zero.*
