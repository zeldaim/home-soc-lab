# Detection Gaps

Known detection gaps and pending improvements in this SOC lab.

---

## T1547.001 — Registry Run Key Persistence (Sysmon EventID 13)

| Item | Detail |
|------|--------|
| Status | ❌ Pending |
| Working Alternative | FIM (rule 750) |
| Priority | High |

### Problem
Sysmon EventID 13 logs are received in `archives.json` but rule 100010 fails to trigger at logtest Phase 3.

### Attempted
- `if_sid 61600` → wrong parent rule (Sysmon base, level 0)
- `if_sid 61615` + `targetObject` pattern → path mismatch (`HKU\SID\...` vs `HKCU\...`)
- `if_sid 61615` + `ruleName: T1547_Run_Key_Create` → Phase 3 not reached

### Current Rule (Non-functional)
```xml
<rule id="100010" level="12">
  <if_sid>61615</if_sid>
  <field name="win.system.eventID">^13$</field>
  <field name="win.eventdata.ruleName">T1547_Run_Key_Create</field>
  <description>T1547 - Registry Run Key Persistence via Sysmon EventID 13</description>
  <mitre>
    <id>T1547.001</id>
  </mitre>
</rule>
```

### Next Steps
- Try direct match without `if_sid` chaining
- Review Wazuh windows_eventchannel rule syntax
- Search community for similar if_sid + level 0 parent issues

---

## T1190 — SQL Injection (OR-based payload)

| Item | Detail |
|------|--------|
| Status | ⚠️ Partial |
| Working | UNION/SELECT based payloads |
| Not Detected | OR-based payloads (`' OR '1'='1`) |

### Problem
Rule 100005 only matches `union`, `select` keywords. OR-based SQLi bypasses detection.

### Next Steps
- Add `+or+`, `%20or%20`, `' or '` patterns to rule 100005


