# SOC Dashboard-4
## Persistence & Lateral Movement

Index:
wazuh-alerts-*

---

# 1) New Scheduled Tasks (4698)

Visualization:
Data Table

KQL:
data.win.system.eventID:4698

Purpose:
Task-based persistence.

Severity:
High

---

# 2) Registry Run/RunOnce Key Modifications (FIM/Registry)

Visualization:
Data Table

KQL:
rule.groups:registry
AND data.win.eventdata.targetObject:(*\\Run\\* OR *\\RunOnce\\*)

Purpose:
Autorun persistence.

Severity:
High

---

# 3) SMB Logon Spike (Type 3)

Visualization:
Line

KQL:
rule.groups:authentication_success
AND data.win.eventdata.logonType:3

Bucket:
Date Histogram @timestamp (5m)

Purpose:
Detect lateral movement burst.

---

# 4) NTLM Logons (Possible Relay / PtH Context)

Visualization:
Data Table

KQL:
data.win.system.eventID:4624
AND data.win.eventdata.authenticationPackageName:NTLM

Columns:
@timestamp
agent.name
data.win.eventdata.targetUserName
data.win.eventdata.ipAddress
data.win.eventdata.workstationName

Purpose:
Highlight NTLM usage for investigation.

Severity:
Medium → High (context dependent)

---

# 5) Service Creation (7045)

Visualization:
Data Table

KQL:
data.win.system.eventID:7045

Columns:
@timestamp
agent.name
data.win.eventdata.serviceName
data.win.eventdata.imagePath
data.win.eventdata.accountName

Purpose:
Persistence / remote execution via service creation.

Severity:
Critical

---

END OF DASHBOARD-4 TEMPLATE
