# SOC Dashboard-5 — Threat Intelligence & IOC Monitoring (Production Template)

**Index pattern:** `wazuh-alerts-*`  
**Time range:** `Last 24 hours` (default; override per investigation)  
**Query language:** **DQL**

> Dashboard-5 purpose: Surface threat intelligence matches, suspicious outbound destinations, malware hash detections, DNS anomalies, and MITRE ATT&CK technique distribution.

---

## Panel 1 — SOC | TI | Threat Intel Matches

**Purpose:** Surface threat intelligence hits from integrated feeds or enrichment pipelines.

**DQL**
```dql
rule.groups:threat_intel
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Threat Intel Hits`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `rule.description`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 20
   - Custom Label: `Detection Rule`

**Columns (display context)**
- `@timestamp`
- `agent.name`
- `data.srcip`
- `data.destip`
- `rule.description`
- `rule.level`

**SOC notes**
- Any hit → validate IOC freshness and asset criticality immediately.
- Multiple hits from the same host → possible active compromise, isolate host.
- High rule.level (≥12) → immediate escalation to Tier 2/3.
- Severity: **High → Critical** (depends on rule.level and asset criticality)

---

## Panel 2 — SOC | TI | Suspicious Outbound Destinations (Non-RFC1918)

**Purpose:** Identify top external destination IPs to detect potential C2 beaconing or data exfiltration.

**DQL**
```dql
rule.groups:firewall AND NOT data.destip:(10.* OR 192.168.* OR 172.16.*)
```

**Visualization:** Horizontal Bar

**Metrics**
- Aggregation: `Count`
- Custom Label: `Outbound Connections`

**Buckets**
- Aggregation: `Terms`
- Field: `data.destip`
- Order by: `Count`
- Order: `Descending`
- Size: 15
- Custom Label: `Destination IP`

**SOC notes**
- High-volume destination → potential C2 beaconing or large data transfer.
- Destinations in sanctioned countries or unexpected regions → geopolitical risk.
- Correlate destination IP with threat intel feeds for known bad actors.
- Severity: **Medium → High** (escalate on TI match or anomalous volume)

---

## Panel 3 — SOC | TI | Malware Hash Detections

**Purpose:** Monitor for known malware hash detections from AV, EDR, or Sysmon enrichment.

**DQL**
```dql
rule.groups:malware
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Malware Detections`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `rule.description`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 20
   - Custom Label: `Detection`

**Columns (display context)**
- `@timestamp`
- `agent.name`
- `file.hash`
- `file.path`
- `rule.description`

**SOC notes**
- Any detection → immediate host isolation assessment.
- Multiple hosts with the same hash → active malware propagation in the network.
- File in system32, temp, or download folders → high-risk execution path.
- Severity: **Critical**

---

## Panel 4 — SOC | TI | DNS Suspicious TLD Watchlist

**Purpose:** Detect DNS queries to suspicious top-level domains (TLDs) often used for malicious infrastructure.

**DQL**
```dql
rule.groups:dns AND data.query:(*.xyz OR *.top OR *.ru OR *.click OR *.gq)
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Suspicious DNS Queries`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.query`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 20
   - Custom Label: `Domain`

3) Sub Aggregation: `Terms`
   - Field: `data.srcip`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 20
   - Custom Label: `Source IP`

**SOC notes**
- Repeated queries to same suspicious domain → possible C2 callback or beaconing.
- High query volume from a single host → DNS tunneling indicator.
- New domain not in the organization's baseline → investigative pivot point.
- Severity: **Medium → High** (context dependent)

---

## Panel 5 — SOC | TI | MITRE ATT&CK Technique Distribution

**Purpose:** Overview of detected MITRE ATT&CK techniques for reporting and coverage analysis.

**DQL**
```dql
rule.mitre.id:*
```

**Visualization:** Pie

**Metrics**
- Aggregation: `Count`
- Custom Label: `Technique Hits`

**Buckets**
- Aggregation: `Terms`
- Field: `rule.mitre.id`
- Order by: `Count`
- Order: `Descending`
- Size: 15
- Custom Label: `MITRE Technique`

**SOC notes**
- Dominant technique → indicates the active phase of an ongoing attack.
- Unexpected technique appearance → new threat vector being exploited.
- Use for weekly/monthly SOC reporting and detection coverage gap analysis.
- Severity: **Low** (informational / reporting)

---

END OF DASHBOARD-5 TEMPLATE
