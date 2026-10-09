# Boundary Conditions & Overlap Prevention

**Scope:** Wazuh SOC Blueprint  
**Standard Version:** 1.1.0  
**Status:** Enforced  

---

## 1. Detection Engineering Boundaries & Constraints

### 1.1 Rule ID Allocation
- Standard Wazuh core rules occupy IDs `1` to `99999`.
- All custom rules in `rules/local_rules.xml` MUST be allocated in the private namespace:
  - `100000–100099`: Threat Intelligence & External Feeds (VirusTotal, MISP, URLhaus)
  - `100100–100199`: DNS & C2 Watchlist Monitoring
  - `100200–100299`: Persistence & Service/Task Creation
  - `100300–100399`: Authentication Abuse & Brute-Force Frequency Thresholds
  - `100400–100499`: Process & Execution Anomalies (Sysmon/LOLBAS)

### 1.2 Windows Auditing & Sysmon Prerequisites
- Advanced Audit Policy Configuration (GPO) must be active for Event IDs:
  - Account Logon: 4624, 4625
  - Privilege Use: 4672
  - Account Management: 4720, 4728, 4732, 4738, 4756
  - Detailed Tracking: 4688 (with Command Line Process Auditing enabled)
  - System Events: 7045 (Service Creation), 4698 (Scheduled Task Creation)
- Sysmon is mandatory for:
  - Event ID 1 (Process Create)
  - Event ID 8 (CreateRemoteThread / Injection)
  - Event ID 10 (ProcessAccess / LSASS)
  - Event ID 25 (ProcessTampering / Injection)

### 1.3 Network Scope & RFC1918 Boundaries
- Internal networks: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.
- Production VPN subnet baseline: `10.212.134.200–210`, `fdff:ffff*`.
- Direct public IP access to RDP (3389) or SMB (445) is treated as a **Tripwire (Expected: 0)**.

---

## 2. Panel Overlap Governance & Taxonomy

An overlap occurs when two or more panels analyze identical log datasets:

| Overlap Type | Technical Criteria | Required Agent Action |
|---|---|---|
| **Full Overlap (Tam Çakışma)** | Identical DQL + Identical Visualization + Identical Bucket | ❌ Prohibited. Remove one or reject the proposal. |
| **Partial Overlap (Kısmi Çakışma)** | Identical DQL base, but differing Visualization or Bucket hierarchy | ⚠️ Justify differentiation; add cross-reference note. |
| **Logical Overlap (Mantıksal Çakışma)** | Differing DQL queries answering the identical investigative security question | ⚠️ Justify; consider merging or consolidating. |
| **Complementary (Tamamlayıcı)** | Shared data source providing orthogonal analytical perspectives (e.g. Trend vs Ranking) | ✅ Permitted. Must link via SOC Notes `Correlate with:`. |

---

## 3. Overlap Detection Algorithm

Before generating any new panel, an agent must execute the following 3-step evaluation:

### Step 1: Query Extraction
Extract core filter constraints from candidate DQL:
- `data.win.system.eventID` (e.g. 4624, 4625, 4698, 7045)
- `rule.groups` (e.g. `authentication_failed`, `threat_intel`, `malware`)
- `data.win.eventdata.logonType` (e.g. 3, 10)
- `data.win.eventdata.authenticationPackageName` (e.g. NTLM)

### Step 2: Dimension & Aggregation Matching
Compare against existing entries in the **Master Panel Registry** (Section 5):
- If `DQL`, `Visualization`, and `Bucket` match exactly → **Full Overlap**.
- If `DQL` matches but `Visualization` or `Bucket` differs → **Partial Overlap**.

### Step 3: Triage & Validation
- Issue appropriate alert template if duplicate.
- If complementary, document the correlation pathway in `SOC notes`.

---

## 4. Known Accepted Architectural Overlaps

1. **D1-P7 vs D4-P3 (Complementary)**:
   - D1-P7: Network Logons Success (Type 3) by `targetUserName` (Horizontal Bar).
   - D4-P3: SMB Logon Spike (Type 3) over time `@timestamp (5m)` (Line Chart).
   - Rationale: D1-P7 provides user attribution; D4-P3 provides temporal burst detection.
2. **D1-P2 vs D2-P2 (Contextual Independence)**:
   - D1-P2: Top Failed Source IPs across all services (`rule.groups:authentication_failed`).
   - D2-P2: Top Failed Source IPs specifically targeting RDP (`eventID:4625`).
   - Rationale: Preserves standalone investigative autonomy for dedicated RDP analysts.
3. **D3-P1 vs D1-P6 (Complementary Correlation)**:
   - D3-P1: Special Privileges Assigned (`4672`).
   - D1-P6: Failed vs Successful Logons.
   - Rationale: PrivEsc analysis requires correlation back to logon origin.
4. **D4-P5 vs D3 General (Categorical Separation)**:
   - D4-P5: Windows Service Creation (`7045`) under Persistence.
   - D3: Group and Account Manipulations under Privilege Escalation.
   - Rationale: Service creation acts primarily as persistence; group membership acts as privilege escalation.

---

## 5. Master Panel Registry (Quick Reference)

| Dashboard | Panel | Event ID / Rule Group | LogonType | Visualization | Primary Bucket Field |
|---|---|---|---|---|---|
| **D1-P1** | KPI Failed Logins | `authentication_failed` | — | Metric | — |
| **D1-P2** | Top Source IP (Failed) | `authentication_failed` | — | Horizontal Bar | `ipAddress` |
| **D1-P3** | Top Target Users (Failed) | `authentication_failed` | — | Vertical Bar | `targetUserName` |
| **D1-P4** | Spike Failed (5m) | `authentication_failed` | — | Line Chart | `@timestamp (5m)` |
| **D1-P5** | Source IP Spike (5m) | `authentication_failed` | — | Bar Chart | `ipAddress` + `@timestamp` |
| **D1-P6** | Failed vs Success | `auth_failed` + `auth_success` | — | Data Table | `targetUserName` + `rule.groups` |
| **D1-P7** | Network Logons Success | `authentication_success` | 3 | Horizontal Bar | `targetUserName` |
| **D2-P1** | RDP Success Users | 4624 | 10 | Horizontal Bar | `targetUserName` |
| **D2-P2** | RDP Failed Source IP | 4625 | — | Horizontal Bar | `ipAddress` |
| **D2-P3** | Auth Logon Type Dist. | 4624 (LT10) OR 4625 | 10 | Data Table | `ipAddress` → `targetUserName` |
| **D2-P4** | RDP Internal Timeline | 4624 | 10 | Vertical Bar (Stacked) | `@timestamp (3h)` + `eventID` |
| **D2-P5** | RDP Internal IP→User→Host | 4624 | 10 | Data Table | `ipAddress` → `targetUserName` → `agent.name` |
| **D2-P6** | RDP VPN IP→User→Host | 4624 | 10 | Data Table | `ipAddress` → `targetUserName` → `agent.name` |
| **D2-P7** | VPN Multi-Host Count | 4624 | — | Data Table | `ipAddress` (Unique Count `agent.name`) |
| **D2-P8** | RDP Public IP Tripwire | 4624 | 10 | Horizontal Bar | `ipAddress` (NOT internal) |
| **D3-P1** | Special Privileges | 4672 | — | Data Table | `agent.name` → `subjectUserName` |
| **D3-P2** | Admin Group Changes | 4728 / 4732 / 4756 | — | Data Table | `agent.name` → `TargetUserName` |
| **D3-P3** | Account Creation | 4720 | — | Data Table | `agent.name` → `targetUserName` |
| **D3-P4** | Source IP Privilege | 4672 / 4728 / 4732 / 4756 | — | Horizontal Bar | `ipAddress` |
| **D4-P1** | Scheduled Tasks | 4698 | — | Data Table | `agent.name` → `taskName` |
| **D4-P2** | Registry Run Keys | `registry` + Run/RunOnce | — | Data Table | `agent.name` → `targetObject` |
| **D4-P3** | SMB Logon Spike | `authentication_success` | 3 | Line Chart | `@timestamp (5m)` |
| **D4-P4** | NTLM Logons | 4624 + NTLM | — | Data Table | `agent.name` → `targetUserName` |
| **D4-P5** | Service Creation | 7045 | — | Data Table | `agent.name` → `serviceName` |
| **D5-P1** | Threat Intel Matches | `threat_intel` | — | Data Table | `agent.name` → `rule.description` |
| **D5-P2** | Suspicious Outbound | `firewall` | — | Horizontal Bar | `data.destip` |
| **D5-P3** | Malware Hash | `malware` | — | Data Table | `agent.name` → `rule.description` |
| **D5-P4** | DNS Suspicious TLD | `dns` | — | Data Table | `agent.name` → `data.query` |
| **D5-P5** | MITRE Distribution | `rule.mitre.id:*` | — | Pie | `rule.mitre.id` |
| **D6-P1** | PowerShell Scripts | 4104 | — | Data Table | `agent.name` → `scriptBlockText` |
| **D6-P2** | Suspicious Process | 4688 / Sysmon 1 | — | Data Table | `agent.name` → `image` |
| **D6-P3** | LSASS Access | Sysmon 10 | — | Data Table | `agent.name` → `sourceImage` |
| **D6-P4** | Process Injection | Sysmon 8 / 25 | — | Data Table | `agent.name` → `targetImage` |
| **D6-P5** | Long Command Lines | 4688 / Sysmon 1 | — | Data Table | `agent.name` → `commandLine` |
