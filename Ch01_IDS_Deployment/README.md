# Ch.01 — IDS Deployment & First Detection

**homelab_AEGIS** · SOC from Scratch · March 22, 2026

---

## Overview

This chapter documents the deployment of a Suricata IDS + Wazuh SIEM stack on a physical homelab environment, followed by two controlled attack simulations: a network reconnaissance scan and an SSH brute force attack. All attacks were detected, logged, and correlated with MITRE ATT&CK techniques.

---

## Lab Environment

| Machine | Role | OS | IP |
|:--------|:-----|:---|:---|
| MacBook Pro (physical) | Management + Wazuh Server | macOS | 172.20.10.4 |
| MacBook VirtualBox → Wazuh OVA | SIEM Server | Ubuntu (OVA) | 172.20.10.9 |
| Kali Laptop (physical) | Attacker | Kali Linux | 172.20.10.8 |
| Kali VirtualBox → aegis-sentinel | Sensor Node | Ubuntu 24.04.4 LTS | 172.20.10.6 |

**Network:** Wireless Access Point · 172.20.10.0/28 · all machines on same subnet

---

## Architecture

```
Kali Laptop (physical) — Attacker (172.20.10.8)
        │
        │  nmap / hydra
        ▼
aegis-sentinel VM (172.20.10.6)
        │
        ├── Suricata IDS  ──►  /var/log/suricata/eve.json
        │                                │
        └── Wazuh Agent  ◄───────────────┘
                │
                │  1514/tcp
                ▼
        Wazuh Server (172.20.10.9)
                │
                ▼
        Wazuh Dashboard → MacBook Pro (172.20.10.4) → browser
```

---

## Stack

| Component | Version | Role |
|:----------|:--------|:-----|
| Suricata | 7.0.3 | Network IDS — packet inspection, rule matching |
| Wazuh Agent | 4.14.4 | Log collector — forwards eve.json to Manager |
| Wazuh Manager | 4.14.4 | SIEM core — rule correlation, alert generation |
| Wazuh Dashboard | 4.14.4 | SOC interface — visualization, threat hunting |
| ET Open Ruleset | — | Suricata signature database |

---

## Chapter Structure

| Folder | Content |
|:-------|:--------|
| [01-Architecture](01-Architecture/) | Lab environment, service status, network diagram |
| [02-Assets](02-Assets/) | Stack, tool versions, key configurations, network table |
| [03-Attack-Scenario](03-Attack_Scenario/) | Attack commands, flag explanations, MITRE mapping |
| [04-Evidences](04-Evidences/) | Wazuh alerts, screenshots, detection analysis |
| [05-LESSONS_LEARNED](05-LESSONS_LEARNED/) | Findings, mitigations, key takeaways |

---

## Phase 1 — Suricata Deployment

Suricata was installed on the aegis-sentinel VM and configured to monitor the `enp0s3` interface in af-packet mode.

**Key configuration (`/etc/suricata/suricata.yaml`):**
```yaml
af-packet:
  - interface: enp0s3
```

ET Open ruleset was pulled with `suricata-update`. Suricata version 7.0.3 started successfully in SYSTEM mode.

Both services confirmed active and enabled at boot:
- `suricata.service` — active (running) since 09:44:29 UTC
- `wazuh-agent.service` — active (running) since 11:02:08 UTC

→ See [01-Architecture](01-Architecture/) for service status screenshots.

---

## Phase 2 — Wazuh Integration

Wazuh all-in-one OVA deployed on MacBook VirtualBox. All three components confirmed running:
- `wazuh-manager.service` — active (running)
- `wazuh-dashboard.service` — active (running) — port 443
- `wazuh-indexer.service` — active (running)

Wazuh Agent installed on aegis-sentinel, registered to Wazuh Manager at `172.20.10.9`. Suricata `eve.json` configured as a log source:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

Agent confirmed active:
```
Agent ID:    001  |  Agent name: Sentinel  |  IP: 172.20.10.6
OS: Ubuntu 24.04.4 LTS  |  Wazuh v4.14.4  |  Status: active
```

→ See [01-Architecture](01-Architecture/) for Wazuh Manager and Dashboard status screenshots.

---

## Phase 3 — Attack Simulation #1: Network Reconnaissance

```bash
sudo nmap -sS -A -T4 172.20.10.6
```

**Result:**
```
PORT   STATE  SERVICE  VERSION
22/tcp open   ssh      OpenSSH 9.6p1 Ubuntu
OS:    Linux 4.15-5.19
```

**Detection:** Suricata detected anomalous ICMP probes from nmap OS fingerprinting.

```
rule.description: Suricata: Alert - SURICATA ICMPv4 unknown code
rule.id:          86601  |  rule.level: 3
data.src_ip:      172.20.10.4  |  data.proto: ICMP  |  data.icmp_code: 9
```

**MITRE ATT&CK:** T1595 — Active Scanning

→ See [03-Attack_Scenario](03-Attack_Scenario/) for full command breakdown.
→ See [04-Evidences](04-Evidences/) for Wazuh alert screenshots.

---

## Phase 4 — Attack Simulation #2: SSH Brute Force

```bash
hydra -l aegis-siem -P /usr/share/wordlists/rockyou.txt -t 4 ssh://172.20.10.6
```

**Result:**
```
[22][ssh] host: 172.20.10.6   login: aegis-siem   password: qwerty
1 of 1 target successfully completed, 1 valid password found
```

**Detection — Brute Force:**

| Level | Rule | Description |
|:------|:-----|:------------|
| 5 | 5760 | `sshd: authentication failed` |
| 8 | 5758 | `Maximum authentication attempts exceeded` |
| 10 | 40111 | `Multiple authentication failures` |
| 10 | 5763 | `sshd: brute force trying to get access to the system` |

**Detection — Initial Access:**

```
rule.description: Multiple authentication failures followed by a success
rule.id:          40112  |  rule.level: 12 (critical)
rule.mitre.id:    T1078, T1110
rule.mitre.tactic: Defense Evasion, Persistence, Privilege Escalation, Initial Access
```

→ See [03-Attack_Scenario](03-Attack_Scenario/) for full Hydra command breakdown.
→ See [04-Evidences](04-Evidences/) for all detection screenshots and alert details.

---

## MITRE ATT&CK Kill Chain

| Phase | Technique | ID | Tool | Detected |
|:------|:----------|:---|:-----|:---------|
| Reconnaissance | Active Scanning | T1595 | nmap -sS -A -T4 | ✅ Suricata ICMPv4 alert |
| Credential Access | Brute Force: Password Guessing | T1110.001 | Hydra + rockyou.txt | ✅ rule 40111, level 10 |
| Lateral Movement | Remote Services: SSH | T1021.004 | Hydra + SSH | ✅ rule 5760, level 5→10 |
| Initial Access | Valid Accounts | T1078 | SSH with stolen creds | ✅ rule 40112, level 12 |

---

## Sentinel — Post-Attack Threat Profile

- **MITRE ATT&CK Top Tactics:** Credential Access (1,716 events), Lateral Movement (1,108 events)
- **Vulnerability Detection:** 58 Critical · 322 High · 887 Medium · 77 Low
- **CIS Ubuntu 24.04 Benchmark:** Score 48% (118 passed, 127 failed)
- **PCI DSS violations:** 10.2.4 (1,720) · 10.2.5 (1,720)

→ See [04-Evidences](04-Evidences/) for full dashboard screenshot.

---

## Key Findings

| # | Finding | Severity | Status |
|:--|:--------|:---------|:-------|
| 1 | Weak SSH password — `qwerty` cracked in under 2 minutes | High | ⚠️ Mitigate |
| 2 | Root SSH login blocked — `PermitRootLogin prohibit-password` | — | ✅ Hardened |
| 3 | IDS in passive mode — attacks detected but not blocked | Medium | ⚠️ Mitigate |
| 4 | Compliance violations auto-flagged — GDPR, PCI DSS, HIPAA, NIST | — | ✅ Working |
| 5 | Dynamic IP instability — DHCP lease renewal broke Agent connectivity | Low | ⚠️ Mitigate |

→ See [05-LESSONS_LEARNED](05-LESSONS_LEARNED/) for full analysis and mitigations.

---

## Next — Ch.02

- Wazuh active response: auto-block attacker IP after N failed attempts
- Custom Suricata rules for lab-specific detection
- fail2ban integration on aegis-sentinel
- MITRE ATT&CK dashboard deep-dive

---

*homelab_AEGIS · github.com/cyb-ersin · Built from scratch, documented from zero.*
