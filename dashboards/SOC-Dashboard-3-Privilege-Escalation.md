# SOC Dashboard-3 — Privilege Escalation & Admin Abuse (Production Template)

**Index pattern:** `wazuh-alerts-*`  
**Time range:** `Last 24 hours` (default; override per investigation)  
**Query language:** **DQL**

> Dashboard-3 purpose: Detect privilege escalation, administrative abuse, special privilege assignments, and privileged group membership tampering.

---

## Panel 1 — SOC | Privilege | Special Privileges Assigned (4672)

**Purpose:** Identify accounts granted sensitive administrative privileges per host.

**DQL**
```dql
data.win.system.eventID:4672
AND data.win.eventdata.subjectUserName:*
AND NOT data.win.eventdata.subjectUserName:*$
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Privilege Assignments`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.subjectUserName`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 15
   - Custom Label: `User`

**SOC notes**
- High frequency of 4672 on non-DC endpoints → investigate for local privilege escalation.
- New or unexpected accounts acquiring SeDebugPrivilege / SeTcbPrivilege → potential exploitation.
- Sudden privilege activity outside normal working hours → correlate with 4624 (Logon) and 4688 (Process Creation).
- Severity: **High** if observed on non-admin baseline user accounts.

---

## Panel 2 — SOC | Privilege | Admin Group Changes (4728/4732/4756)

**Purpose:** Identify accounts added to privileged security groups (Domain Admins, Administrators).

**DQL**
```dql
data.win.system.eventID:(4728 OR 4732 OR 4756)
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Group Membership Changes`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 15
   - Custom Label: `User Added`

**SOC notes**
- Any addition to Domain Admins or Enterprise Admins → immediate escalation and verification with change management.
- User added to local Administrators on endpoints → possible local persistence.
- Repeated membership modifications → potential adversary staging elevated access.
- Severity: **Critical** (High for local groups, Critical for domain groups).

---

## Panel 3 — SOC | Privilege | Account Creation (4720)

**Purpose:** Detect newly created accounts which may indicate persistence or unauthorized staging.

**DQL**
```dql
data.win.system.eventID:4720
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Account Creations`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 20
   - Custom Label: `New Account`

**SOC notes**
- Unapproved account creation → isolate and verify provisioning ticket.
- Creation of generic naming accounts (e.g. `support`, `tempadmin`) → attacker evasion tactic.
- Correlate with:
  - Event 4672 (Special Privileges Assigned)
  - Events 4728/4732/4756 (Immediate addition to administrative groups)
- Severity: **High → Critical** if account created by non-standard administrator.

---

## Panel 4 — SOC | Privilege | Host & Subject Breakdown (4672/4728/4732/4756)

**Purpose:** Correlate privilege assignment and group changes across endpoints and executing subject users.

**DQL**
```dql
data.win.system.eventID:(4672 OR 4728 OR 4732 OR 4756)
AND data.win.eventdata.subjectUserName:*
AND NOT data.win.eventdata.subjectUserName:*$
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Privilege Events`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 15
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.subjectUserName`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 15
   - Custom Label: `Executing User (Subject)`

3) Sub Aggregation: `Terms`
   - Field: `data.win.system.eventID`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 5
   - Custom Label: `Event ID`

**SOC notes**
- Windows Security events 4672 and 4728/4732 do not generate a native ipAddress field.
- To trace network origin, correlate `data.win.eventdata.subjectLogonId` with contemporaneous Event 4624 (Logon) events.
- Concentrated privilege activity on workstations indicates active lateral movement and local privilege escalation.
- Severity: **High**

---

END OF DASHBOARD-3 TEMPLATE
