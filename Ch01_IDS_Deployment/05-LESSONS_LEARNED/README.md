# 05 — Lessons Learned

## What Worked

**Suricata → Wazuh pipeline** functioned end-to-end. Every attack generated alerts that propagated from `eve.json` through the Wazuh Agent to the Manager and appeared in the Dashboard within seconds.

**Wazuh rule escalation** mirrored real SOC triage: individual failed logins (level 5) escalated automatically to brute force pattern (level 10) and finally to initial access (level 12). No manual correlation needed.

**MITRE ATT&CK auto-mapping** — every alert was pre-tagged with ATT&CK tactic, technique, and sub-technique. Compliance frameworks (GDPR, PCI DSS, HIPAA, NIST 800-53) were flagged automatically on every brute force alert.

---

## Findings & Mitigations

### Finding 1 — Weak SSH Password

**Issue:** User `aegis-siem` had password `qwerty` — position ~491 in rockyou.txt. Cracked in under 2 minutes with `-t 4`.

**Mitigation:**
- Enforce strong password policy (minimum 12 chars, mixed case, special characters)
- Switch to SSH key authentication — makes brute force attacks ineffective
- Disable password authentication entirely in `/etc/ssh/sshd_config`:
```
PasswordAuthentication no
```

### Finding 2 — Root SSH Login Correctly Blocked

**Observation:** `PermitRootLogin prohibit-password` was configured — root brute force attempts failed as expected.

**Status:** ✅ Correctly hardened default. No action required.

### Finding 3 — IDS in Passive Mode

**Issue:** Suricata runs in IDS mode — `data.alert.action: allowed` on all alerts. Attacks were detected but not blocked. An attacker who knows the IDS is passive can operate freely once detected.

**Mitigation:**
- Configure Wazuh active response to auto-block attacker IP after N failed attempts
- Or switch Suricata to IPS (inline) mode for active blocking
- Implement `fail2ban` as a lightweight alternative

### Finding 4 — Compliance Violations Auto-Flagged

**Observation:** Every brute force alert was automatically mapped to GDPR, PCI DSS, HIPAA, and NIST 800-53. In a real organization, this level of automated compliance monitoring would significantly reduce manual reporting overhead.

### Finding 5 — Dynamic IP Instability

**Issue:** Running the lab on a wireless access point caused occasional IP lease renewals, breaking Wazuh Agent → Manager connectivity during the session.

**Mitigation:**
- Use a stable router (Fritz!Box) instead of mobile hotspot for lab sessions
- Configure static IPs via netplan on all VMs to prevent disruption on DHCP lease renewal
- Or use Wazuh Agent's hostname-based registration instead of IP

---

## Key Takeaways

**On detection:** Suricata alone sees the network. Wazuh alone sees the system. Together they provide full SOC visibility — network-level events (port scans, protocol anomalies) correlated with host-level events (failed logins, PAM activity, process execution).

**On attacker behavior:** A password in the top 500 of rockyou.txt is effectively no password. The brute force took under 2 minutes. Any credential in a common wordlist should be considered compromised.

**On noise vs signal:** Not every alert is a threat. The APT user-agent alert from `apt update` was informational — Ubuntu's package manager generating HTTP traffic to `archive.ubuntu.com`. SOC analysts must understand baseline behavior to distinguish noise from signal.

**On compliance:** Security incidents automatically map to legal obligations. A successful brute force is not just a technical event — it is a GDPR IV_35.7.d, PCI DSS 10.2.4, and HIPAA 164.312.b violation simultaneously.

---

## Next — Ch.02

- Wazuh active response: auto-block attacker IP after N failed attempts (T1110 mitigation)
- Custom Suricata rules for lab-specific detection patterns
- `fail2ban` integration on aegis-sentinel
- MITRE ATT&CK dashboard deep-dive and coverage analysis
- Vulnerability Detection review — 58 Critical CVEs identified on aegis-sentinel

---

*homelab_AEGIS · Ch.01 complete · github.com/cyb-ersin*
