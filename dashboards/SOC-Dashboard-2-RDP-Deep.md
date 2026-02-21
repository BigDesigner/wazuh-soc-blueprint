# SOC Dashboard-2
## RDP Deep Monitoring

Index:
wazuh-alerts-*

Time Range:
Last 24h (Adjust per incident)

---

# 1) RDP | Success Logons (Type 10) | Top Users

Visualization:
Horizontal Bar

KQL:
rule.groups:authentication_success
AND data.win.eventdata.logonType:10
AND NOT data.win.eventdata.targetUserName: *$

Buckets:
Y: data.win.eventdata.targetUserName
Size: 15
Order: Count Desc

Purpose:
Identify active RDP users.

---

# 2) RDP | Failed Logons (Type 10) | Top Source IP

Visualization:
Horizontal Bar

KQL:
rule.groups:authentication_failed
AND data.win.eventdata.logonType:10

Buckets:
Y: data.win.eventdata.ipAddress
Size: 15
Order: Count Desc

Purpose:
Detect brute-force over RDP.

---

# 3) RDP | Fail → Success Pattern (Same IP)

Visualization:
Data Table

KQL:
rule.groups:(authentication_failed OR authentication_success)
AND data.win.eventdata.logonType:10

Buckets:
Split Rows:
data.win.eventdata.ipAddress
data.win.eventdata.targetUserName
rule.groups

Metric:
Count

Purpose:
Credential compromise detection.

Severity:
High if fail followed by success.

---

# 4) RDP | External IP Logons

Visualization:
Data Table

KQL:
rule.groups:authentication_success
AND data.win.eventdata.logonType:10
AND NOT data.win.eventdata.ipAddress:(10.* OR 192.168.* OR 172.16.*)

Columns:
@timestamp
agent.name
data.win.eventdata.targetUserName
data.win.eventdata.ipAddress

Purpose:
Detect internet-origin RDP logons.

Severity:
Critical if admin account.

---

# 5) RDP | Logon Time Analysis (Off-Hours)

Visualization:
Line

KQL:
rule.groups:authentication_success
AND data.win.eventdata.logonType:10

Bucket:
Date Histogram @timestamp (1h)

Purpose:
Identify abnormal hour activity.

Recommended Overlay:
Business hours baseline

---

END OF DASHBOARD-2 TEMPLATE
