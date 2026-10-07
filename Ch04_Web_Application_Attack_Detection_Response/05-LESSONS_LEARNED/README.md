# 05 — Lessons Learned

## Key Findings

| # | Finding | Status |
|:--|:--------|:-------|
| 1 | SQLi authentication bypass produced an admin session | ✅ Proven |
| 2 | Suricata had HTTP visibility but no relevant SQLi alert initially | ⚠️ Detection gap |
| 3 | Custom Suricata SID `1000001` detected the tested bypass | ✅ Resolved |
| 4 | Wazuh received and parsed the custom Suricata alert | ✅ Confirmed |
| 5 | Kali DHCP address changed after reboot | ✅ Rule source generalized to `any` |
| 6 | Wazuh Agent can be active while temporarily disconnected from Manager | ✅ Troubleshot |
| 7 | Attacker-specific containment blocked TCP/3000 | ✅ Confirmed |
| 8 | Legitimate client still received HTTP 200 | ✅ Service continuity preserved |
| 9 | IP blocking does not remove the SQLi root cause | ✅ Documented |

---

## Errors & Solutions

### Error 1 — Custom Rule Did Not Fire After Syntax Test

**Symptom:**
`suricata -T` succeeded, but the next attack produced no custom alert.

**Cause:**
`suricata -T` validates configuration syntax only. It does not reload the running Suricata service.

**Solution:**

```bash
sudo systemctl restart suricata
```

After restart, the same request produced SID `1000001`.

---

### Error 2 — Fixed Attacker IP Made the Rule Fragile

**Symptom:**
After reboot, Kali no longer used `192.168.178.200`.

**Cause:**
DHCP assigned `192.168.178.50`, while the first rule version only matched `.200`.

**Solution:**
Generalize the source:

```text
any any -> 192.168.178.185 3000
```

while keeping the destination, endpoint and payload conditions specific.

---

### Error 3 — Suricata Alert Present, Wazuh Initially Missing New Events

**Symptom:**
The custom alert existed in `eve.json`, but the Wazuh dashboard showed only older events.

**Cause:**
The Wazuh Agent temporarily lost connectivity to the Manager on `192.168.178.127:1514`.

**Evidence:**

```text
Unable to connect to '[192.168.178.127]:1514/tcp'
```

followed later by:

```text
Connected to the server ([192.168.178.127]:1514/tcp)
```

**Solution:**
Restore Manager availability / connectivity, then repeat a fresh detection test.

---

## Key Takeaways

**Visibility is not detection.**
The malicious login request was already present in Suricata HTTP telemetry before any useful SQLi alert existed.

**Noise is not coverage.**
Suricata produced other generic HTTP anomaly alerts, but those did not mean the successful authentication bypass had been detected.

**Detection engineering starts with a measurable gap.**
The lab first proved the exploit and then proved the absence of a matching alert. The custom rule closed that specific gap.

**Sensor-first troubleshooting is efficient.**
Checking `eve.json` first separated Suricata detection problems from Wazuh forwarding / SIEM problems.

**Containment and remediation are different.**
Blocking `192.168.178.50` stopped the current source, but it did not fix the SQL injection vulnerability.

**The real permanent fix belongs in the application.**
In a production environment the SQL query / authentication logic would need parameterized queries, server-side validation, secure session handling and post-fix testing. Juice Shop was intentionally not patched because it is a training target.

**Containment should minimize impact.**
Kali timed out while `core` still received HTTP 200. The service was not unnecessarily taken offline for every client.

---

## Project AEGIS — Final Takeaway

AEGIS started with basic IDS visibility and ended with a complete security workflow:

```text
Attack
  ↓
Telemetry
  ↓
Detection
  ↓
Investigation
  ↓
Containment
  ↓
Validation
```

The most important learning outcome was not a single tool. It was understanding how Burp, Suricata, Wazuh, packet/log evidence and firewall response connect during one incident.

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.04 — Web Application Attack, Detection & Incident Response*
