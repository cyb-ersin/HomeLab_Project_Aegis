# 02 — Assets

## Tool Versions

| Tool | Version | Machine |
|:-----|:--------|:--------|
| Suricata | 7.0.3 | aegis-sentinel |
| Wazuh Agent | 4.14.4 | aegis-sentinel |
| Wazuh Manager | 4.14.4 | Wazuh OVA |
| Wazuh Dashboard | 4.14.4 | Wazuh OVA |
| iptables | 1.8.10-3ubuntu2 | aegis-sentinel |
| fail2ban | 1.0.x | aegis-sentinel |
| Hydra | 9.6 | Kali Laptop |

---

## Key Configurations

### Wazuh Active Response — ossec.conf (Wazuh Manager)

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5763</rules_id>
  <timeout>300</timeout>
</active-response>
```

![ossec.conf Active Response](Bildschirmfoto%202026-03-24%20um%2009.23.56.png)

---

### Custom Suricata Rule — local.rules (aegis-sentinel)

```
alert tcp any any -> 172.20.10.6 22 (msg:"AEGIS SSH Brute Force Attempt"; flow:to_server; threshold:type threshold, track by_src, count 5, seconds 60; sid:9000001; rev:1;)
```

**suricata.yaml — rule-files:**

```yaml
rule-files:
  - suricata.rules
  - local.rules
```

![suricata.yaml local.rules](Bildschirmfoto%202026-03-24%20um%2011.22.46.png)

---

### fail2ban SSH Jail — jail.local (aegis-sentinel)

```ini
[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 5
bantime  = 300
findtime = 60
```

![jail.local config](Bildschirmfoto%202026-03-24%20um%2012.27.05.png)

---

### Wazuh Agent Log Sources — ossec.conf (aegis-sentinel)

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/syslog</location>
</localfile>
```

---

### fail2ban Custom Rule — local_rules.xml (Wazuh Manager)

```xml
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
