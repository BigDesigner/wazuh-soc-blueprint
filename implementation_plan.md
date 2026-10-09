# Implementation Plan: MITRE Detection Gap Remediation (T1070 & T1486)

**Plan ID:** `PLAN-2026-10-09-MITRE-GAPS`  
**Target:** `rules/local_rules.xml`, `mitre/MITRE-Coverage-Matrix.md`, `.specs/boundary-conditions.md`  
**Standard Enforced:** `.specs/constitution.md` (v1.1.0) & `.specs/boundary-conditions.md` (v1.1.0)  
**Status:** Hardened & Ready for Execution  

---

## 1. Objectives & Scope
Close the two highest-priority MITRE ATT&CK detection gaps identified in `.memory-bank/bugs/bug-list.md` and `.tasks/pipeline.md`:
1. **GAP-002 / T1070.001 (Indicator Removal):** Windows Security and System Event log clearing detection (Event IDs 1102 & 104).
2. **GAP-001 / T1486 & T1490 (Data Encrypted for Impact & Inhibit System Recovery):** Ransomware volume shadow copy deletion and high-frequency mass-file modification alerting.

---

## 2. Technical Implementation Specifications

### 2.1 Update `rules/local_rules.xml`

#### Group: `defense_evasion,log_clearing`
- **Rule 100500 (Level 12 — Critical):**
  - **Condition:** Windows Security Audit Log Cleared (Event ID 1102).
  - **Syntax:**
    ```xml
    <rule id="100500" level="12">
      <if_sid>60000</if_sid>
      <field name="win.system.eventID">^1102$</field>
      <group>defense_evasion,indicator_removal,log_clearing</group>
      <description>Defense Evasion: Windows Security Audit log was cleared on $(win.system.computer) by $(win.eventdata.subjectUserName).</description>
      <mitre>
        <id>T1070.001</id>
      </mitre>
    </rule>
    ```
- **Rule 100501 (Level 10 — High):**
  - **Condition:** Windows Event Log File Cleared (Event ID 104 - System/Application).
  - **Syntax:**
    ```xml
    <rule id="100501" level="10">
      <if_sid>60000</if_sid>
      <field name="win.system.eventID">^104$</field>
      <group>defense_evasion,indicator_removal,log_clearing</group>
      <description>Defense Evasion: Windows Event Log file was cleared on $(win.system.computer) (Log Channel: $(win.system.channel)).</description>
      <mitre>
        <id>T1070.001</id>
      </mitre>
    </rule>
    ```

#### Group: `impact,ransomware`
- **Rule 100600 (Level 13 — Critical):**
  - **Condition:** Inhibit System Recovery via Shadow Copy Deletion (vssadmin / wmic / wbadmin / bcdedit).
  - **Syntax:**
    ```xml
    <rule id="100600" level="13">
      <if_sid>60000</if_sid>
      <field name="win.system.eventID">^(4688|1)$</field>
      <field name="win.eventdata.commandLine">vssadmin.*delete.*shadows|wmic.*shadowcopy.*delete|wbadmin.*delete.*catalog|bcdedit.*recoveryenabled.*no</field>
      <group>impact,ransomware,defense_evasion</group>
      <description>Ransomware Behavior: Volume shadow copy deletion or recovery inhibition command executed ($(win.eventdata.commandLine)).</description>
      <mitre>
        <id>T1486</id>
        <id>T1490</id>
      </mitre>
    </rule>
    ```
- **Rule 100601 (Level 12 — Critical — Frequency Threshold):**
  - **Condition:** Mass File Modifications detected by Wazuh FIM (Syscheck).
  - **Syntax:**
    ```xml
    <rule id="100601" level="12" frequency="100" timeframe="60">
      <if_matched_group>syscheck</if_matched_group>
      <same_location />
      <group>impact,ransomware,file_modification</group>
      <description>Potential Ransomware Activity: High-frequency file modifications (100+ changes in 60s) detected by FIM on $(agent.name).</description>
      <mitre>
        <id>T1486</id>
      </mitre>
    </rule>
    ```

---

### 2.2 Update `.specs/boundary-conditions.md`
- Register the new rule ID allocations in Section 1.1:
  - `100500–100599`: Defense Evasion & Indicator Removal (T1070.001)
  - `100600–100699`: Impact & Ransomware Invalidation (T1486, T1490)

### 2.3 Update `mitre/MITRE-Coverage-Matrix.md`
- Elevate **T1070.001** and **T1486** from "Identified Gaps" to the "Detailed Technique Mapping" table with `Status: ✅ Full`.
- Update tactic coverage level for **Defense Evasion** and **Impact** in the Coverage Summary table.

### 2.4 Update Memory Bank
- Close `GAP-001` and `GAP-002` in `.memory-bank/bugs/bug-list.md`.
- Update `.tasks/pipeline.md` marking `TASK-002` complete.

---

## 3. Verification Protocol
1. **XML Syntax Validation:** Execute Python `xml.etree.ElementTree` validation script against `rules/local_rules.xml`.
2. **Tag Validity Check:** Confirm all Wazuh correlation tags (`<same_location />`, `<if_matched_group>`) comply with OSSEC/Wazuh schema rules.
3. **MITRE Alignment:** Verify technique IDs `T1070.001`, `T1486`, `T1490` conform to standard MITRE Enterprise ATT&CK matrix v14+.

---

### 🛡️ Audit Notes (Hardening Applied by Sentinel Plan-Audit)

1. **Wazuh Schema Safety Control:** Corrected proposed `<same_agent />` tag to valid OSSEC correlation tag `<same_location />`. OSSEC/Wazuh parser rejects `<same_agent />` with an invalid XML tag error during engine startup.
2. **Regex Engine Compatibility:** Formatted command-line regex to match standard OS_Regex syntax without unescaped PCRE2 dependencies, ensuring universal compatibility across Wazuh Manager deployments.
3. **Multi-Technique MITRE Mapping:** Added secondary MITRE ID `<id>T1490</id>` (Inhibit System Recovery) to Rule 100600 alongside `T1486` for comprehensive matrix accuracy.
4. **Boundary Namespace Alignment:** Formatted rule IDs strictly within the private `100500+` and `100600+` namespaces defined in `.specs/boundary-conditions.md`.
5. **Anti-Eager Execution Gate:** Modification of `rules/local_rules.xml` remains strictly blocked until human confirmation is granted.
