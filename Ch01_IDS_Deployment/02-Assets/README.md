# 02 — Assets

## Stack

| Component | Version | Role |
|:----------|:--------|:-----|
| Suricata | 7.0.3 | Network IDS — packet inspection, ET Open rule matching |
| Wazuh Agent | 4.14.4 | Log collector — forwards eve.json to Manager |
| Wazuh Manager | 4.14.4 | SIEM core — rule correlation, alert generation |
| Wazuh Indexer | 4.14.4 | OpenSearch-based storage and search |
| Wazuh Dashboard | 4.14.4 | SOC interface — Threat Hunting, MITRE ATT&CK, compliance |
| ET Open Ruleset | — | Suricata community signature database |
| Hydra | 9.6 | Attacker — SSH brute force tool |
| Nmap | 7.98 | Attacker — network scanner |
| rockyou.txt | — | Wordlist — 14,344,399 passwords |

---

## Key Configuration

### Suricata — Interface (`/etc/suricata/suricata.yaml`)

```yaml
af-packet:
  - interface: enp0s3
```

Suricata monitors `enp0s3` in af-packet mode. Output: `fast.log` and `eve.json`.

### Wazuh Agent — Suricata Integration (`/var/ossec/etc/ossec.conf`)

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

Wazuh Agent reads Suricata's JSON output and forwards all events to the Manager.

---

## Network

| Host | IP | Subnet | Gateway |
|:-----|:---|:-------|:--------|
| aegis-sentinel (sensor) | 172.20.10.6 | /28 | 172.20.10.1 |
| Wazuh Server | 172.20.10.9 | /28 | 172.20.10.1 |
| Kali (attacker) | 172.20.10.8 | /28 | 172.20.10.1 |
| MacBook Pro (management) | 172.20.10.4 | /28 | 172.20.10.1 |

**Subnet:** 172.20.10.0/28 — max 14 hosts — Wireless Access Point
