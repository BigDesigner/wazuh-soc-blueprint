# Implementation Plan: SOC Dashboards Comprehensive Remediation & Standardization

**Plan ID:** `PLAN-2026-10-09-SOC-AUDIT`  
**Target:** Dashboards 1 through 6 (`dashboards/`)  
**Standard Enforced:** `.specs/constitution.md` (v1.1.0) & `.specs/boundary-conditions.md` (v1.1.0)  
**Status:** Hardened & Ready for Execution  

---

## 1. Objectives & Scope
Remediate all defects, DQL errors, missing SOC notes, and query redundancies identified during the comprehensive dashboard audit (`.memory-bank/audits/audit-7583260.md`). Ensure 100% compliance with production standards before designing new dashboards.

---

## 2. Phased Execution Roadmap

### Phase 1: Critical Logic & DQL Repairs (P0)

#### 1.1 Fix D6-P5 Fatal DQL Syntax Error (`SOC-Dashboard-6-Execution-Process.md`)
- **Issue:** `commandLine.length > 500` is illegal in OpenSearch DQL and causes a fatal parse error.
- **Remediation:** 
  - Replace illegal script evaluation with a production-safe DQL wildcard query:
    ```dql
    (data.win.system.eventID:4688 OR data.win.system.eventID:1)
    AND data.win.eventdata.commandLine:????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????*
    ```
  - Alternatively, prepare Wazuh Rule `100400` in `rules/local_rules.xml` using regex length matching `.{500,}` and query `(rule.id:100400 OR (data.win.system.eventID:(4688 OR 1) AND data.win.eventdata.commandLine:??????????*))`.
  - Add missing trailing separator `---` before template footer.

#### 1.2 Fix D3-P4 Nonexistent Field Dependency (`SOC-Dashboard-3-Privilege-Escalation.md`)
- **Issue:** Event IDs 4672, 4728, 4732, and 4756 do not generate `data.win.eventdata.ipAddress`. Query currently returns 0 events.
- **Remediation:**
  - Pivot panel to **Target Host & Actor Matrix** (Host vs Subject User):
    - **DQL:** `data.win.system.eventID:(4672 OR 4728 OR 4732 OR 4756) AND data.win.eventdata.subjectUserName:* AND NOT data.win.eventdata.subjectUserName:*$`
    - **Visualization:** Data Table
    - **Buckets:**
      1) `agent.name` (Host)
      2) `data.win.eventdata.subjectUserName` (Actor User)
      3) `data.win.system.eventID` (Privilege Event Type)
  - Retain `## Panel 4 — SOC | Privilege | Host & Subject Breakdown` with actionable SOC notes.

#### 1.3 Fix D2-P4 Event Scope Mismatch (`SOC-Dashboard-2-RDP-Deep.md`)
- **Issue:** Title specifies `(4624/4625)` and split series aggregates by `eventID`, but DQL hardcodes `data.win.system.eventID:4624`.
- **Remediation:**
  - Update DQL to include both successful RDP and failed attempts:
    ```dql
    (
      (data.win.system.eventID:4624 AND data.win.eventdata.logonType:10)
      OR
      (data.win.system.eventID:4625)
    )
    AND data.win.eventdata.ipAddress:(10.38.1.* OR 172.16.16.*)
    AND NOT data.win.eventdata.targetUserName:*$
    ```

---

### Phase 2: Elimination of Duplications & Dashboard-2 Overhaul (P1)

#### 2.1 Eliminate Accidental Duplicate in D2-P2
- **Issue:** D2-P2 omits `logonType:10`, capturing all Windows failed logons and duplicating D1-P2.
- **Remediation:**
  - Restrict DQL to RDP failed logons:
    ```dql
    data.win.system.eventID:4625
    AND data.win.eventdata.logonType:10
    AND data.win.eventdata.ipAddress:*
    AND NOT data.win.eventdata.targetUserName:*$
    ```
  - Add SOC notes cross-referencing D1-P2.

#### 2.2 Standardize Dashboard-2 Structure & Complete SOC Notes
- **Issue:** Missing standard header, section labels are singular (`**Metric**`, `**Bucket**`), and 0 out of 8 panels have `**SOC notes**` or `Severity`.
- **Remediation:**
  - Update Header to: `# SOC Dashboard-2 — Deep RDP Monitoring (Production Template)` with `Time range: Last 24 hours`.
  - Normalize section labels to `**Metrics**` and `**Buckets**`.
  - Author detailed triage notes and severity ratings for all 8 panels (P1 to P8) incorporating thresholds, operational baselines, and cross-references.

---

### Phase 3: Alignment, Formatting & Field Name Fixes (P2)

#### 3.1 Standardize Dashboard-3 Headings & Casing
- Change `# Panel 1..4` to `## Panel 1..4`.
- Replace emoji `1️⃣ Split Rows` with standard `1) Aggregation: Terms`.
- Normalize field casing: Change `data.win.eventdata.TargetUserName` to `data.win.eventdata.targetUserName` (lowercase).

#### 3.2 Dual-Schema Support in D6-P2 (Windows 4688 vs Sysmon Event 1)
- Expand process fields so queries match both Sysmon (`image`, `parentImage`) and Windows native auditing (`newProcessName`, `parentProcessName`):
  ```dql
  (data.win.system.eventID:4688 OR data.win.system.eventID:1)
  AND (data.win.eventdata.parentImage:(*winword.exe OR *excel.exe OR *powershell.exe OR *cmd.exe) OR data.win.eventdata.parentProcessName:(*winword.exe OR *excel.exe OR *powershell.exe OR *cmd.exe))
  AND (data.win.eventdata.image:(*whoami.exe OR *net.exe OR *nltest.exe OR *vssadmin.exe) OR data.win.eventdata.newProcessName:(*whoami.exe OR *net.exe OR *nltest.exe OR *vssadmin.exe))
  ```

#### 3.3 Correct Field Drift in Dashboard 5 (P3 & P4)
- **Panel 3 (Malware):** Replace ECS `file.hash` and `file.path` in columns list with Wazuh native fields: `syscheck.sha256`, `syscheck.path`, and `data.virustotal.sha1`.
- **Panel 4 (DNS):** Update DQL to support both generic and Sysmon Event 22 fields:
  ```dql
  rule.groups:dns
  AND (data.query:(*.xyz OR *.top OR *.ru OR *.click OR *.gq) OR data.win.eventdata.queryName:(*.xyz OR *.top OR *.ru OR *.click OR *.gq))
  ```

---

## 3. Verification & Validation Protocol
1. **DQL Grammar Check:** Inspect all modified DQL queries against OpenSearch Dashboards parser specifications.
2. **Constitution Compliance Audit:** Verify every modified panel conforms to the 8-section mandatory hierarchy.
3. **Boundary Registry Sync:** Update `.specs/boundary-conditions.md` master panel registry with corrected panel metadata (specifically D2-P2 and D3-P4).

---

### 🛡️ Audit Notes (Hardening Applied by Sentinel Plan-Audit)

1. **Security & Data Integrity Control (D3-P4):** Prevented retention of broken `data.win.eventdata.ipAddress` filter. Validated against Windows Event Log specifications to ensure analysts are not left with permanently empty panels.
2. **OpenSearch Engine Safety Control (D6-P5):** Replaced `.length > 500` script call that would crash OpenSearch index queries with a syntax-compliant DQL wildcard pattern.
3. **Telemetry Parity Control (D6-P2):** Hardened process execution query against environments that do not have Sysmon deployed, ensuring native Windows Event ID 4688 (`parentProcessName` / `newProcessName`) is actively monitored.
4. **Boundary Condition Enforcement (D2-P2):** Eliminated query overlap between D1-P2 and D2-P2 by strictly locking D2-P2 to `logonType:10`.
5. **No Fluff & No Code Premature Execution:** In compliance with Anti-Eager Execution, no dashboard files were modified during this plan-audit phase. All execution remains gated on human confirmation.
