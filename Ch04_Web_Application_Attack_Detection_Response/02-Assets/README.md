# 02 — Assets

## Tools

| Tool | Machine | Role |
|:-----|:--------|:-----|
| Burp Suite Community Edition | `h4des` | HTTP interception, inspection and replay |
| OWASP Juice Shop | `edge` | Intentionally vulnerable web application |
| Docker | `edge` | Hosts Juice Shop on TCP/3000 |
| Suricata 7.0.3 | `edge` | Network telemetry + custom detection |
| ET Open rules | `edge` | Baseline Suricata ruleset |
| Wazuh Agent | `edge` | Forwards Suricata JSON + host logs |
| Wazuh Manager / Dashboard | `wazuh-server` | Central alerting and investigation |
| iptables | `edge` | Incident containment |
| curl | `h4des`, `core` | Connectivity / service validation |

---

## Suricata Rule Path

Verified configuration:

```yaml
default-rule-path: /var/lib/suricata/rules

rule-files:
  - suricata.rules
  - local.rules
```

Custom rules are stored separately from the update-managed ET ruleset:

```text
/var/lib/suricata/rules/local.rules
```

---

## Custom SQLi Rule

```text
alert http any any -> 192.168.178.185 3000 (msg:"AEGIS SQLi Login Bypass Attempt"; flow:established,to_server; http.method; content:"POST"; http.uri; content:"/rest/user/login"; http.request_body; content:"admin@juice-sh.op'--"; classtype:web-application-attack; sid:1000001; rev:1;)
```

### Rule Elements

| Element | Meaning |
|:--------|:--------|
| `alert http` | Alert on matching HTTP traffic |
| `any any -> 192.168.178.185 3000` | Any source to Juice Shop |
| `flow:established,to_server` | Established client → server flow |
| `http.method; content:"POST"` | Login POST request |
| `http.uri; content:"/rest/user/login"` | Login endpoint |
| `http.request_body; content:"admin@juice-sh.op'--"` | Tested SQLi payload |
| `sid:1000001` | Local custom signature ID |

---

## Suricata Validation

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```

Expected result:

```text
Configuration provided was successfully loaded.
```

After changing rules:

```bash
sudo systemctl restart suricata
```

Check the custom alert:

```bash
sudo grep -F 'AEGIS SQLi Login Bypass Attempt' /var/log/suricata/eve.json | tail -n 3
```

---

## Wazuh Input

`edge` collects the Suricata JSON file with:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

The resulting Suricata alerts appear in Wazuh under generic Suricata rule:

```text
rule.id: 86601
```

---

## Containment Rule

```bash
sudo iptables -I DOCKER-USER 1 -i wlp3s0 -s 192.168.178.50 -p tcp --dport 3000 -j DROP
```

Validation:

```bash
sudo iptables -L DOCKER-USER -n --line-numbers
```

Attacker test:

```bash
curl --max-time 5 -I http://192.168.178.185:3000
```

Legitimate client test from `core`:

```bash
curl --max-time 5 -I http://192.168.178.185:3000
```

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.04 — Web Application Attack, Detection & Incident Response*
