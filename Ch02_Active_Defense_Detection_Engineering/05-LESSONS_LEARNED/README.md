# 05 — Lessons Learned

## Key Findings

| # | Finding | Severity | Status |
|:--|:--------|:---------|:-------|
| 1 | Ch.01 gap closed — IDS now actively blocks attackers | — | ✅ Resolved |
| 2 | fail2ban log format incompatible with Wazuh pre-decoder by default | Medium | ✅ Resolved |
| 3 | Suricata rule path mismatch — `/etc/suricata/rules/` vs `/var/lib/suricata/rules/` | Medium | ✅ Resolved |
| 4 | Custom Wazuh rules must be defined on Manager, not Agent | — | ✅ Documented |
| 5 | Two independent block layers provide defense-in-depth | — | ✅ Operational |
| 6 | sudo activity post-login generates highest alert volume (288) | Low | ⚠️ Monitor |

---

## Errors Encountered & Solutions

### Error 1 — Suricata Rule Not Triggering

**Symptom:** Custom rule `sid:9000001` written to `/etc/suricata/rules/local.rules` but no alerts in eve.json.

**Root Cause:** Suricata's `default-rule-path` in `suricata.yaml` points to `/var/lib/suricata/rules/`. Writing a rule to `/etc/suricata/rules/` without copying it to the correct path results in silent failure.

**Diagnosis:**
```bash
sudo grep -i "local.rules" /var/log/suricata/suricata.log
# Output: No rule files match the pattern /var/lib/suricata/rules/local.rules
```

**Solution:**
```bash
sudo cp /etc/suricata/rules/local.rules /var/lib/suricata/rules/local.rules
sudo systemctl restart suricata
```

**Verification:**
```bash
sudo tail -20 /var/log/suricata/suricata.log
# Output: 1 rules successfully loaded
```

---

### Error 2 — fail2ban Logs Not Visible in Wazuh Dashboard

**Symptom:** fail2ban banning IPs correctly but no alerts appearing in Wazuh Dashboard.

**Root Cause:** fail2ban's default log target (`/var/log/fail2ban.log`) writes without a hostname field:
```
2026-03-24 10:06:00,595 fail2ban.actions [15817]: NOTICE [sshd] Ban 172.20.10.3
```
Wazuh pre-decoder expects `timestamp hostname program[pid]: message`. Missing hostname = pre-decoder fails silently.

**Solution:** Redirect fail2ban output to syslog:
```
# /etc/fail2ban/fail2ban.conf
logtarget = SYSLOG
```
Syslog writes with hostname included — pre-decoder correctly identifies `program_name: fail2ban.actions`.

---

### Error 3 — Wazuh Manager Failing to Start (Decoder XML Syntax Error)

**Symptom:**
```
wazuh-analysisd: ERROR: (1452): Syntax error on regex
wazuh-analysisd: CRITICAL: Configuration error at 'etc/decoders/local_decoder.xml'
```

**Root Cause:** Custom decoder XML used regex characters `\[` and `\]` which conflicted with XML parsing.

**Solution:** Decoder was removed entirely. Since `program_name: fail2ban.actions` is already correctly identified by the pre-decoder, a separate decoder is unnecessary — the rule alone is sufficient.

---

### Error 4 — Custom Rule Added to Agent, Not Working

**Symptom:** `local_rules.xml` added to aegis-sentinel (Agent) but no alerts in Dashboard.

**Root Cause:** Rule matching happens on the **Manager**, not the Agent. The Agent only collects and forwards logs.

**Solution:** Rule file moved to Wazuh OVA (Manager) at `/var/ossec/etc/rules/local_rules.xml`.

---

## Mitigations

| Finding | Mitigation |
|:--------|:-----------|
| Weak SSH password (`qwerty`) | Change to strong password or disable password auth entirely — use SSH keys |
| Active Response timeout (300s) | In production, increase to 3600s+ or implement permanent ban with analyst review |
| sudo alert volume high | Expected in lab — in production, filter noise with alert tuning |
| DHCP instability (Wazuh Agent IP) | Assign static IP to aegis-sentinel to prevent agent connectivity drops |

---

## Key Takeaways

**Defense in Depth works.** Two independent blocking layers (Wazuh Active Response + fail2ban) ensure that if one fails, the other continues protecting the host.

**Log format matters.** Wazuh's pre-decoder has strict format expectations. A missing hostname field in fail2ban's default log output silently broke the entire pipeline — understanding log parsing is a critical SOC skill.

**Rule matching is Manager-side.** Agents collect and forward — all correlation and rule matching happens on the Manager. This distinction is fundamental to Wazuh's architecture.

**Always verify with logtest.** `wazuh-logtest` is the fastest way to validate decoder and rule behavior without running a live attack:
```bash
sudo /var/ossec/bin/wazuh-logtest
```

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.02 — Active Defense & Detection Engineering*
