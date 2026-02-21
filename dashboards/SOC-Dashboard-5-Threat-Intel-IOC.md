# SOC Dashboard-5
## Threat Intelligence & IOC Monitoring

Index:
wazuh-alerts-*

---

# 1) Threat Intel Matches (Generic)

Visualization:
Data Table

KQL:
rule.groups:threat_intel

Columns:
@timestamp
agent.name
data.srcip
data.destip
rule.description
rule.level

Purpose:
Surface TI hits from feeds/enrichment.

Severity:
High → Critical (depends on rule.level and asset criticality)

---

# 2) Suspicious Outbound Destinations (Non-RFC1918)

Visualization:
Horizontal Bar

KQL:
rule.groups:firewall
AND NOT data.destip:(10.* OR 192.168.* OR 172.16.*)

Buckets:
Y: data.destip
Size: 15
Order: Count Desc

Purpose:
Identify top external destinations.

---

# 3) Malware Hash Detections (If Integrated)

Visualization:
Data Table

KQL:
rule.groups:malware

Columns:
@timestamp
agent.name
file.hash
file.path
rule.description

Purpose:
Hash-based detections (EDR/AV/Sysmon pipelines).

Severity:
Critical

---

# 4) DNS Suspicious TLD Watchlist

Visualization:
Data Table

KQL:
rule.groups:dns
AND data.query:(*.xyz OR *.top OR *.ru OR *.click OR *.gq)

Columns:
@timestamp
agent.name
data.query
data.srcip

Purpose:
TLD-based hunting pivot.

Severity:
Medium → High (context dependent)

---

# 5) MITRE ATT&CK Technique Distribution

Visualization:
Pie

KQL:
rule.mitre.id:*

Bucket:
rule.mitre.id
Size: 15

Purpose:
Technique distribution overview for reporting.

---

END OF DASHBOARD-5 TEMPLATE
