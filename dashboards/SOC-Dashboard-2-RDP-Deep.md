# SOC Dashboard-2 — RDP Deep Monitoring (Production)

**Index pattern:** `wazuh-alerts-*`  
**Time range:** `Last 7 days`  
**Query language:** **DQL**

> Dashboard-2 purpose: RDP behavior + VPN-derived access visibility + public exposure tripwire.  
> SSL VPN logs are not ingested; VPN detection is based on the VPN IP pool.

---

## Panel 1 — SOC | RDP | Success | Top Users

**Purpose:** Identify active RDP users (baseline + anomalies).

**DQL**
```dql
data.win.system.eventID:4624
AND data.win.eventdata.logonType:10
AND data.win.eventdata.targetUserName:*
AND NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Horizontal Bar  
**Metrics**
- Aggregation: `Count`
- Custom Label: `Successful Logons`

**Buckets**
- Aggregation: `Terms`
- Field: `data.win.eventdata.targetUserName`
- Order by: `Count`
- Order: `Descending`
- Size: `15`
- Custom Label: `User Account`

---

## Panel 2 — SOC | RDP | Failed | Top Source IP

**Purpose:** Detect brute-force / password guessing over RDP.

**DQL**
```dql
data.win.system.eventID:4625
AND data.win.eventdata.ipAddress:*
AND NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Horizontal Bar  
**Metrics**
- Aggregation: `Count`
- Custom Label: `Failed Attempts`

**Buckets**
- Aggregation: `Terms`
- Field: `data.win.eventdata.ipAddress`
- Order by: `Count`
- Order: `Descending`
- Size: `15`
- Custom Label: `Source IP`

---

## Panel 3 — SOC | Authentication | Logon Type Distribution

**Purpose:** Single table view for RDP Success (4624+LogonType10) and Auth Fail (4625), with IPv6 hygiene.

**DQL**
```dql
(
  (data.win.system.eventID:4624 AND data.win.eventdata.logonType:10)
  OR
  (data.win.system.eventID:4625)
)
AND data.win.eventdata.ipAddress:*
AND NOT data.win.eventdata.targetUserName:*$
AND NOT data.win.eventdata.ipAddress:("::1")
AND NOT data.win.eventdata.ipAddress:(fe80* OR 2001*)
```

**Visualization:** Data Table  
**Metrics**
- Aggregation: `Count`
- Custom Label: `Attempts`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `data.win.eventdata.ipAddress`
   - Order: `Descending`
   - Custom Label: `Source IP`

2) Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Order: `Descending`
   - Custom Label: `Target User`

3) Aggregation: `Terms`
   - Field: `data.win.system.eventID`
   - Order: `Descending`
   - Custom Label: `Event ID`

---

## Panel 4 — SOC | RDP | Internal | Timeline (4624/4625)

**Purpose:** Internal RDP activity timeline (success/fail trend).

**DQL**
```dql
data.win.system.eventID:4624
```

**Visualization:** Vertical Bar (Timeline)  
**Metrics**
- Aggregation: `Count`
- Custom Label: `Events`

**Buckets**
- X-axis:
  - Aggregation: `Date Histogram`
  - Field: `@timestamp`
  - Interval: `3h`
  - Custom Label: `Time (3h)`

- Split series:
  - Aggregation: `Terms`
  - Field: `data.win.system.eventID`
  - Order: `Descending`
  - Size: `10`
  - Custom Label: `Event ID`

> Note: This panel’s DQL is intentionally kept as configured in the dashboard snapshot.  
> If you want strict “Internal RDP” scoping, use Panel-5’s internal filter as the base and add `eventID:(4624 OR 4625)`.

---

## Panel 5 — SOC | RDP | Internal | IP → User → Host

**Purpose:** Internal visibility drilldown: source IP → user → destination host (agent).

**DQL**
```dql
data.win.system.eventID:4624
```

**Visualization:** Data Table  
**Metrics**
- Aggregation: `Count`
- Custom Label: `Count`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `data.win.eventdata.ipAddress`
   - Order: `Descending`
   - Custom Label: `Source IP`

2) Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Order: `Descending`
   - Custom Label: `Target User`

3) Aggregation: `Terms`
   - Field: `agent.name`
   - Order: `Descending`
   - Custom Label: `Host`

---

## Panel 6 — SOC | RDP | Success | SSL VPN Users (IP → User → Host)

**Purpose:** VPN pool → user mapping for RDP success (investigation view).  
(SSL VPN logs are not ingested; identification is by VPN IP pool.)

**DQL**
```dql
data.win.system.eventID:4624
AND data.win.eventdata.logonType:10
AND data.win.eventdata.ipAddress:*
AND NOT data.win.eventdata.targetUserName:*$
AND (
  data.win.eventdata.ipAddress:(10.212.134.200 OR 10.212.134.201 OR 10.212.134.202 OR 10.212.134.203 OR 10.212.134.204 OR 10.212.134.205 OR 10.212.134.206 OR 10.212.134.207 OR 10.212.134.208 OR 10.212.134.209 OR 10.212.134.210)
  OR data.win.eventdata.ipAddress:"fdff:ffff*"
)
```

**Visualization:** Data Table  
**Metrics**
- Aggregation: `Count`
- Custom Label: `RDP Success`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `data.win.eventdata.ipAddress`
   - Order: `Descending`
   - Custom Label: `VPN Source IP`

2) Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Order: `Descending`
   - Custom Label: `Target User`

3) Aggregation: `Terms`
   - Field: `data.win.eventdata.logonType`
   - Order: `Descending`
   - Custom Label: `Logon Type`

---

## Panel 7 — SOC | SSL VPN | Multi-Host Access Count

**Purpose:** Lateral signal: same VPN IP touching multiple destination hosts (distinct `agent.name`).

**DQL**
```dql
data.win.system.eventID:4624
AND data.win.eventdata.ipAddress:*
AND NOT data.win.eventdata.targetUserName:*$
AND (
  data.win.eventdata.ipAddress:(10.212.134.200 OR 10.212.134.201 OR 10.212.134.202 OR 10.212.134.203 OR 10.212.134.204 OR 10.212.134.205 OR 10.212.134.206 OR 10.212.134.207 OR 10.212.134.208 OR 10.212.134.209 OR 10.212.134.210)
  OR data.win.eventdata.ipAddress:"fdff:ffff*"
)
```

**Visualization:** Data Table  
**Metrics**
- Aggregation: `Unique Count`
- Field: `agent.name`
- Custom Label: `Distinct Target Hosts`

**Buckets**
- Aggregation: `Terms`
- Field: `data.win.eventdata.ipAddress`
- Order by: `Distinct Target Hosts`
- Order: `Descending`
- Size: `20`
- Custom Label: `VPN Source IP`

---

## Panel 8 — SOC | RDP | Success | Public IP (Tripwire)

**Purpose:** Tripwire for direct public-origin RDP success (should be **0** in a hardened environment).

**DQL**
```dql
data.win.system.eventID:4624
```

**Visualization:** Horizontal Bar  
**Metrics**
- Aggregation: `Count`
- Custom Label: `RDP Success`

**Buckets**
- Aggregation: `Terms`
- Field: `data.win.eventdata.ipAddress`
- Order by: `Count`
- Order: `Descending`
- Size: `20`
- Custom Label: `Public Source IP`

---

END OF DASHBOARD-2 TEMPLATE
