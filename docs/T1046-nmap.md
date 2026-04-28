# T1046 — Network Service Scanning (Nmap)

## Overview

| Item | Detail |
|------|--------|
| MITRE Technique | [T1046 Network Service Scanning](https://attack.mitre.org/techniques/T1046/) |
| Attacker VM | Kali Linux |
| Target VM | Rocky Linux 9 |
| Detection | Wazuh custom rule |
| Response | Alert → Kibana dashboard |

---

## Attack

```bash
# Kali Linux
nmap -sS -p 1-1000 <Rocky_IP>
```

---

## Detection Rule

Wazuh detects multiple connection attempts in a short window via the syslog decoder and a custom frequency rule.

```xml
<rule id="100001" level="10" frequency="10" timeframe="10">
  <if_matched_sid>110</if_matched_sid>
  <match>NMAP</match>
  <description>Possible Nmap port scan detected (T1046)</description>
  <mitre>
    <id>T1046</id>
  </mitre>
</rule>
```

---

## Validation

1. Run Nmap from Kali
2. Confirm alert appears in Wazuh Dashboard
3. Verify event visible on Kibana SOC dashboard

---

## Lessons Learned

- `frequency` + `timeframe` combination is key for scan detection
- Tuning threshold prevents false positives from legitimate network tools
