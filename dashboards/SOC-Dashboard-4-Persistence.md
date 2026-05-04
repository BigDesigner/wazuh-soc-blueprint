# SOC Dashboard-4 — Persistence & Lateral Movement (Production Template)

**Index pattern:** `wazuh-alerts-*`  
**Time range:** `Last 24 hours` (default; override per investigation)  
**Query language:** **DQL**

> Dashboard-4 purpose: Detect persistence mechanisms (scheduled tasks, registry autorun, service creation) and lateral movement indicators (SMB spikes, NTLM relay/PtH).

---

## Panel 1 — SOC | Persistence | Scheduled Tasks (4698)

**Purpose:** Detect creation of new scheduled tasks which is a common persistence mechanism.

**DQL**
```dql
data.win.system.eventID:4698
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Scheduled Tasks Created`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.taskName`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 20
   - Custom Label: `Task Name`

**SOC notes**
- New task creation on unexpected host → investigate for persistence.
- Off-hours task creation → highly suspicious, correlate with 4672/4688.
- Verify task creator (subjectUserName) against baseline.
- Severity: **High**

---

## Panel 2 — SOC | Persistence | Registry Run/RunOnce Keys

**Purpose:** Monitor modifications to Registry Run and RunOnce keys used for malware autostart.

**DQL**
```dql
rule.groups:registry AND data.win.eventdata.targetObject:(*\\Run\\* OR *\\RunOnce\\*)
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Registry Modifications`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetObject`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 20
   - Custom Label: `Registry Key`

**SOC notes**
- New Run/RunOnce entry → potential malware persistence.
- Correlate with process creation (4688/Sysmon) for the binary path.
- Unknown binary paths or temp directory locations → investigate immediately.
- Severity: **High**

---

## Panel 3 — SOC | Lateral | SMB Logon Spike (Type 3, 5m)

**Purpose:** Detect sudden bursts of SMB authentication which may indicate automated lateral movement or worm propagation.

**DQL**
```dql
rule.groups:authentication_success 
AND data.win.eventdata.logonType:3 
AND data.win.eventdata.targetUserName:* 
AND NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Line Chart

**Metrics**
- Aggregation: `Count`
- Custom Label: `SMB Logon Events`

**Buckets**
- X-axis:
  - Aggregation: `Date Histogram`
  - Field: `@timestamp`
  - Interval: `5m`
  - Custom Label: `Time (5m)`

**SOC notes**
- Sudden spike → possible automated lateral movement burst or scanning.
- Flat elevated baseline → potential worm propagation or misconfigured service.
- Correlate with Dashboard-1 Panel 7 (user-level SMB baseline).
- Severity: **High**

---

## Panel 4 — SOC | Lateral | NTLM Logons (Relay / PtH Context)

**Purpose:** Identify NTLM authentication usage to detect potential NTLM Relay or Pass-the-Hash (PtH) attacks.

**DQL**
```dql
data.win.system.eventID:4624 
AND data.win.eventdata.authenticationPackageName:NTLM 
AND data.win.eventdata.targetUserName:* 
AND NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `NTLM Logon Events`

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
   - Custom Label: `User`

3) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.ipAddress`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 20
   - Custom Label: `Source IP`

**Columns (display context)**
- `@timestamp`
- `agent.name`
- `data.win.eventdata.targetUserName`
- `data.win.eventdata.ipAddress`
- `data.win.eventdata.workstationName`

**SOC notes**
- NTLM in Kerberos-only environment → suspicious relay attempt.
- Same IP authenticating to multiple hosts via NTLM → Pass-the-Hash indicator.
- Unexpected NTLM from server segment → investigate immediately.
- Severity: **Medium → High** (context dependent)

---

## Panel 5 — SOC | Persistence | Service Creation (7045)

**Purpose:** Detect creation of new Windows services which can be used for persistence or remote command execution.

**DQL**
```dql
data.win.system.eventID:7045
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Services Created`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.serviceName`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 20
   - Custom Label: `Service Name`

**Columns (display context)**
- `@timestamp`
- `agent.name`
- `data.win.eventdata.serviceName`
- `data.win.eventdata.imagePath`
- `data.win.eventdata.accountName`

**SOC notes**
- Unknown service binary path → possible backdoor persistence.
- Service running as SYSTEM → high-risk persistence or privilege escalation.
- Service from temp/download folders → immediate investigation required.
- Correlate with 4697 (Service Installed event).
- Severity: **Critical**

---

END OF DASHBOARD-4 TEMPLATE
