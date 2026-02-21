# SOC Dashboard-3
## Privilege Escalation Monitoring

Index:
wazuh-alerts-*

---

# 1) Admin Group Membership Changes

Visualization:
Data Table

KQL:
data.win.system.eventID:(4728 OR 4732 OR 4756)

Columns:
@timestamp
agent.name
data.win.eventdata.memberName
data.win.eventdata.targetUserName

Purpose:
User added to privileged group.

Severity:
Critical

---

# 2) Special Privileges Assigned (Event 4672)

Visualization:
Horizontal Bar

KQL:
data.win.system.eventID:4672

Buckets:
Y: data.win.eventdata.subjectUserName
Size: 15

Purpose:
Privilege token assignments.

Severity:
High

---

# 3) Service Installation (Event 4697)

Visualization:
Data Table

KQL:
data.win.system.eventID:4697

Columns:
@timestamp
agent.name
data.win.eventdata.serviceName
data.win.eventdata.subjectUserName

Purpose:
Persistence via service install.

Severity:
High

---

# 4) UAC Bypass Indicators

Visualization:
Data Table

KQL:
data.win.system.eventID:4688
AND data.win.eventdata.newProcessName:(*fodhelper.exe OR *eventvwr.exe)

Purpose:
Common UAC bypass methods.

Severity:
High

---

# 5) Suspicious Process Spawn by Admin

Visualization:
Data Table

KQL:
data.win.system.eventID:4688
AND data.win.eventdata.subjectUserName:(*admin*)

Columns:
@timestamp
agent.name
data.win.eventdata.newProcessName
data.win.eventdata.parentProcessName

Purpose:
Abnormal process under admin context.

Severity:
Medium → High

---

END OF DASHBOARD-3 TEMPLATE
