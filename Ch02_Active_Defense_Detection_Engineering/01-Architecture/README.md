# 01 — Architecture

## Lab Environment

| Machine | Role | OS | IP |
|:--------|:-----|:---|:---|
| MacBook Pro (physical) | Management + Wazuh Dashboard | macOS | 172.20.10.4 |
| MacBook VirtualBox → Wazuh OVA | SIEM Server (Manager + Dashboard) | Ubuntu (OVA) | 172.20.10.9 |
| Kali Laptop (physical) | Attacker | Kali Linux | 172.20.10.3 |
| Kali VirtualBox → aegis-sentinel | Sensor + Defense Node | Ubuntu 24.04.4 LTS | 172.20.10.6 |

**Network:** Wireless Access Point · 172.20.10.0/28 · all machines on same subnet

---

## Network Diagram

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
        │                           │
        │                    Active Response
        │                    firewall-drop ──► iptables DROP 172.20.10.3
        │
        └── fail2ban ──► syslog ──► Wazuh Agent ──► Manager
                                            rule 100100 (T1110)
```

---

## Service Status

### Wazuh Agent — aegis-sentinel

![Wazuh Agent Active](Bildschirmfoto%202026-03-24%20um%2009.03.57.png)

### Wazuh Manager — Wazuh OVA

![Wazuh Manager Active](Bildschirmfoto%202026-03-24%20um%2009.26.54.png)

### Suricata IDS — aegis-sentinel

![Suricata Active](Bildschirmfoto%202026-03-24%20um%2011.24.13.png)

---

## Stack

| Component | Version | Role |
|:----------|:--------|:-----|
| Suricata | 7.0.3 | Network IDS |
| Wazuh Agent | 4.14.4 | Log collector |
| Wazuh Manager | 4.14.4 | SIEM core |
| Wazuh Dashboard | 4.14.4 | SOC interface |
| iptables | 1.8.10 | Firewall backend |
| fail2ban | 1.0.x | Independent ban layer |
