# SOC Dashboard-3 — Privilege Escalation & Admin Abuse (Production Template)

**Index pattern:** `wazuh-alerts-*`  
**Time range:** `Last 24 hours` (default; expand during investigation)  
**Query language:** **DQL**

> Dashboard-3 purpose: Detect privilege escalation, admin abuse, special privilege assignments, and group membership changes.

---

# Panel 1 — SOC | Special Privileges Assigned

**Purpose:** Identify accounts granted special privileges per host.

**DQL**
```dql
data.win.system.eventID:4672
AND data.win.eventdata.subjectUserName:*
AND NOT data.win.eventdata.subjectUserName:*$
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Privilege Assignments (4672)`

**Buckets**

1️⃣ Split Rows  
- Aggregation: `Terms`
- Field: `agent.name`
- Order by: `Count`
- Order: `Descending`
- Size: `10`
- Custom Label: `Host`

2️⃣ Split Rows  
- Aggregation: `Terms`
- Field: `data.win.eventdata.subjectUserName`
- Order by: `Count`
- Order: `Descending`
- Size: `15`
- Custom Label: `User`

**SOC notes**
- High frequency on a single host → investigate.
- New or unexpected accounts → possible escalation.
- Sudden privilege activity outside baseline → review 4624 and 4688 correlation.
- Severity: **High** if non-admin baseline user.


---

# Panel 2 — SOC | Admin Group Membership Changes

**Purpose:** Identify accounts added to privileged groups (potential privilege escalation or persistence).

**DQL**
```dql
data.win.system.eventID:(4728 OR 4732 OR 4756)
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Group Membership Changes`

**Buckets**

1️⃣ Split Rows  
- Aggregation: `Terms`
- Field: `agent.name`
- Order by: `Count`
- Order: `Descending`
- Size: `10`
- Custom Label: `Host`

2️⃣ Split Rows  
- Aggregation: `Terms`
- Field: `data.win.eventdata.TargetUserName`
- Order by: `Count`
- Order: `Descending`
- Size: `15`
- Custom Label: `User Added`

**SOC notes**
- Any addition to Administrators or Domain Admins → immediate review.
- Unexpected user additions → possible privilege escalation.
- Repeated group changes → persistence attempt.
- Severity: **High**

---

## Panel 3 — SOC | Group Changes | Admin Groups Modified

**Purpose:** Detect group membership changes (4728, 4732, 4756).

**DQL**
```dql
data.win.system.eventID:(4728 OR 4732 OR 4756)
```

**Visualization:** Vertical Bar

**Metrics**
- Aggregation: `Count`
- Custom Label: `Group Modifications`

**Buckets**
- Aggregation: `Terms`
- Field: `data.win.eventdata.targetUserName`
- Order by: `Count`
- Order: `Descending`
- Size: `15`
- Custom Label: `Affected Account`

**SOC notes**
- User added to Domain Admins or Administrators → Critical.
- Validate change ticket.
- Severity: **Critical** if privileged group.

---

## Panel 4 — SOC | Account Created with Elevated Rights

**Purpose:** Detect new account creation (4720) combined with privilege activity.

**DQL**
```dql
data.win.system.eventID:4720
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Account Creations`

**Buckets**
- Split rows:
  - Aggregation: `Terms`
  - Field: `data.win.eventdata.targetUserName`
  - Size: `20`
  - Custom Label: `New Account`

**SOC notes**
- New admin accounts are high risk.
- Correlate with 4672 and group modifications.
- Severity: **High → Critical**.

---

## Panel 5 — SOC | Privilege Escalation Timeline (5m)

**Purpose:** Detect escalation burst patterns.

**DQL**
```dql
data.win.system.eventID:(4672 OR 4728 OR 4732 OR 4756)
```

**Visualization:** Line Chart

**Metrics**
- Aggregation: `Count`
- Custom Label: `Escalation Events`

**Buckets**
- X-axis:
  - Aggregation: `Date Histogram`
  - Field: `@timestamp`
  - Interval: `5m`
  - Custom Label: `Time (5m)`

**SOC notes**
- Spike pattern → automated privilege abuse.
- Combine with authentication anomalies.
- Severity: **High**.

---

## Panel 6 — SOC | Source IP | Privilege Activity

**Purpose:** Identify source systems triggering escalation events.

**DQL**
```dql
data.win.system.eventID:(4672 OR 4728 OR 4732 OR 4756)
AND data.win.eventdata.ipAddress:*
```

**Visualization:** Horizontal Bar

**Metrics**
- Aggregation: `Count`
- Custom Label: `Events`

**Buckets**
- Aggregation: `Terms`
- Field: `data.win.eventdata.ipAddress`
- Order by: `Count`
- Order: `Descending`
- Size: `15`
- Custom Label: `Source IP`

**SOC notes**
- Internal workstation → possible compromised endpoint.
- Domain Controller only → expected (validate).
- Severity: **High** if unusual source.

---

END OF DASHBOARD-3 TEMPLATE
