# T1190 — Exploit Public-Facing Application (SQL Injection)

## Overview

| Item | Detail |
|------|--------|
| MITRE Technique | [T1190 Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/) |
| Attacker VM | Kali Linux |
| Target VM | Rocky Linux 9 (Apache) |
| Detection | Wazuh custom rule 100005 + rule chaining |
| Response | Active Response — iptables block (timeout 300s) |

---

## Attack

```bash
# Kali Linux — SQLi payload via curl
curl "http://<Rocky_IP>/index.php?id=1' OR '1'='1"
```

---

## Detection Rules

### Base rule — single SQLi attempt

```xml
<rule id="100005" level="6">
  <if_group>web_appsec</if_group>
  <url_match>SELECT|UNION|INSERT|DROP|OR '1'='1</url_match>
  <description>SQL Injection attempt detected (T1190)</description>
  <mitre>
    <id>T1190</id>
  </mitre>
</rule>
```

### Chained rule — 5+ attempts from same IP within 60s → escalate

```xml
<rule id="100006" level="12" frequency="5" timeframe="60">
  <if_matched_sid>100005</if_matched_sid>
  <same_source_ip/>
  <description>Repeated SQL Injection from same IP (T1190) — blocked</description>
  <mitre>
    <id>T1190</id>
  </mitre>
</rule>
```

---

## Active Response

Triggered on rule 100006 — blocks attacker IP via iptables for 300 seconds.

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>100006</rules_id>
  <timeout>300</timeout>
</active-response>
```

---

## Validation

1. Send 5+ SQLi payloads from Kali within 60s
2. Confirm level 12 alert in Wazuh Dashboard
3. Confirm iptables rule added on Rocky:
   ```bash
   iptables -L -n | grep <Kali_IP>
   ```
4. Re-attempt attack from Kali — connection dropped
5. After 300s — rule automatically removed

---

## Lessons Learned

- Rule chaining (frequency + same_source_ip) reduces false positives significantly
- `overwrite="yes"` required when modifying parent rules with level 0
- Active Response timeout prevents permanent blocks from misconfigured rules
