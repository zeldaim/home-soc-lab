# T1110 — Brute Force (SSH)

## Overview

| Item | Detail |
|------|--------|
| MITRE Technique | [T1110 Brute Force](https://attack.mitre.org/techniques/T1110/) |
| Attacker VM | Kali Linux |
| Target VM | Rocky Linux 9 |
| Detection | Wazuh rule chaining — 5 failed logins within 60s |
| Response | Active Response — iptables block (timeout 300s) |

---

## Attack

```bash
# Kali Linux — SSH brute-force via Hydra
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://<Rocky_IP>
```

---

## Detection Rules

### Base rule — single SSH failure (Wazuh built-in)

Wazuh built-in rule `5710` detects individual SSH authentication failures.

### Chained rule — 5+ failures from same IP within 60s → escalate

```xml
<rule id="100010" level="10" frequency="5" timeframe="60">
  <if_matched_sid>5710</if_matched_sid>
  <same_source_ip/>
  <description>SSH brute-force detected — 5+ failures in 60s (T1110)</description>
  <mitre>
    <id>T1110</id>
  </mitre>
</rule>
```

---

## Active Response

Triggered on rule 100010 — blocks attacker IP via iptables for 300 seconds.

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>100010</rules_id>
  <timeout>300</timeout>
</active-response>
```

---

## Validation

1. Run Hydra from Kali
2. Confirm level 10 alert in Wazuh Dashboard after 5th failure
3. Confirm iptables block on Rocky:
   ```bash
   iptables -L -n | grep <Kali_IP>
   ```
4. Confirm subsequent SSH attempts from Kali are dropped
5. After 300s — block automatically removed

---

## Lessons Learned

- Chaining on Wazuh built-in `5710` avoids duplicating existing detection logic
- `same_source_ip` is critical — without it, distributed brute-force from multiple IPs won't trigger the chained rule
- Active Response logs visible at `/var/ossec/logs/active-responses.log`
