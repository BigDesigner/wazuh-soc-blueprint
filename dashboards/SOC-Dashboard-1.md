# SOC Dashboard-1
## Authentication & Correlation (Production Template)

Index Pattern:
wazuh-alerts-*

Time Range:
Last 24 Hours (Default)
Override allowed per investigation.

---

# 1) SOC | KPI | Failed Logins

Visualization:
Metric

KQL:
rule.groups:authentication_failed

Metric:
Count

Purpose:
Global authentication failure indicator.

SOC Interpretation:
- Sudden jump = brute / spray attempt
- Correlate with Source IP panel

Severity Guidance:
Medium → High if spike detected

---

# 2) SOC | Top | Source IP (Failed Logins)

Visualization:
Horizontal Bar

KQL:
rule.groups:authentication_failed
AND NOT data.win.eventdata.targetUserName: *$

Buckets:
Y (Terms):
data.win.eventdata.ipAddress
Size: 15
Order: Count Desc

Purpose:
Top attacking IP addresses.

SOC Interpretation:
- Public IP → external brute force
- Internal IP → lateral movement risk

Action:
Reputation check + firewall review

Severity:
High if repeated

---

# 3) SOC | Top | Target Users (Failed Logins)

Visualization:
Vertical Bar

KQL:
rule.groups:authentication_failed
AND NOT data.win.eventdata.targetUserName: *$

Buckets:
X (Terms):
data.win.eventdata.targetUserName
Size: 15
Order: Count Desc

Purpose:
Most targeted accounts.

SOC Interpretation:
- Admin accounts targeted → escalation attempt
- Single user heavy targeting → password spray

Severity:
Medium → High (admin targeted)

---

# 4) SOC | Spike | Failed Logins (5m)

Visualization:
Line Chart

KQL:
rule.groups:authentication_failed

Buckets:
X (Date Histogram):
@timestamp
Interval: 5m

Y:
Count

Purpose:
Detect brute-force timing pattern.

Detection Logic:
- 5 min > 50 events = probable brute wave

SOC Interpretation:
- Flat continuous = credential stuffing
- Sharp spike = password spray burst

Severity:
High

---

# 5) SOC | Source IP Spike (5m > 15 fails)

Visualization:
Bar Chart (Split by IP)

KQL:
rule.groups:authentication_failed

Buckets:
Split Series (Terms):
data.win.eventdata.ipAddress
Size: 10

Sub-bucket:
Date Histogram
Interval: 5m

Metric:
Count

Operational Threshold:
Manual review if IP > 15 failures in 5m

Purpose:
Identify concentrated attack source.

Severity:
High
Critical if external IP + admin targeting

---

# 6) SOC | Correlation | Failed vs Success (Users)

Visualization:
Data Table

KQL:
rule.groups:(authentication_failed OR authentication_success)

Buckets:
Split Rows (Terms):
data.win.eventdata.targetUserName
Size: 20

Split Rows (Terms):
rule.groups

Metric:
Count

Purpose:
User-based authentication behavior correlation.

SOC Interpretation:
- Many fails + later success → possible credential compromise
- Success only → normal behavior
- Fail only → brute attempt

Severity:
Medium → High (fail→success pattern)

---

# 7) SOC | Network Logons | Success (Users Only)

Visualization:
Horizontal Bar

KQL:
rule.groups:authentication_success
AND data.win.eventdata.logonType:3

Buckets:
Y (Terms):
data.win.eventdata.targetUserName
Size: 15
Order: Count Desc

Purpose:
Successful SMB / network logons.

SOC Interpretation:
- Sudden rise → lateral movement
- Service accounts dominating → expected
- User accounts spike → suspicious

Severity:
Medium
High if unusual workstation pivot

---

END OF DASHBOARD-1 TEMPLATE
