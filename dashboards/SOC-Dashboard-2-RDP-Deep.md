# SOC Dashboard 2 — RDP Deep (Production)

**Index pattern:** `wazuh-alerts-*`  
**Query language:** **DQL**  
**Goal:** RDP behavior + internal RDP visibility + VPN-derived RDP usage + public exposure tripwire.

---

## Panel 1 — SOC | RDP | Success | Top Users

**Purpose:** RDP (LogonType 10) successful logons by user (baseline + anomalies).

**DQL**
```dql
data.win.system.eventID:4624 AND
data.win.eventdata.logonType:10 AND
data.win.eventdata.targetUserName:* AND
NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Horizontal Bar  
**Metric**
- Aggregation: `Count`
- Custom Label: `Successful Logons`

**Bucket**
- Aggregation: `Terms`
- Field: `data.win.eventdata.targetUserName`
- Order by: `Count`
- Order: `Descending`
- Size: `15`
- Custom Label: `User`

Sub Aggregation: ❌ None

---

## Panel 2 — SOC | RDP | Failed | Top Source IP

**Purpose:** Failed logons by source IP (credential guessing visibility).

**DQL**
```dql
data.win.system.eventID:4625 AND
data.win.eventdata.ipAddress:* AND
NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Horizontal Bar  
**Metric**
- Aggregation: `Count`
- Custom Label: `Failed Attempts`

**Bucket**
- Aggregation: `Terms`
- Field: `data.win.eventdata.ipAddress`
- Order by: `Count`
- Order: `Descending`
- Size: `15`
- Custom Label: `Source IP`

Sub Aggregation: ❌ None

---

## Panel 3 — SOC | Authentication | Logon Type Distribution

**Purpose:** Combined view of RDP Success (4624+LogonType10) and Auth Fail (4625), excluding loopback/IPv6 noise.

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
**Metric**
- Aggregation: `Count`
- Custom Label: `Attempts`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `data.win.eventdata.ipAddress`
   - Size: `20`
   - Order: `Descending`
   - Custom Label: `Source IP`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Size: `20`
   - Order: `Descending`
   - Custom Label: `Target User`

3) Sub Aggregation: `Terms`
   - Field: `data.win.system.eventID`
   - Size: `10`
   - Order: `Descending`
   - Custom Label: `Event ID`

---

## Panel 4 — SOC | RDP | Internal | Timeline (4624/4625)

**Purpose:** Internal subnet RDP activity timeline (trend/baseline).

**DQL**
```dql
data.win.system.eventID:4624 AND data.win.eventdata.logonType:10 AND data.win.eventdata.ipAddress:(10.38.1.* OR 172.16.16.*) AND NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Vertical Bar  
Mode: **Stacked**

**Metric**
- Aggregation: `Count`
- Custom Label: `Internal RDP Events`

**X-Axis**
- Aggregation: `Date Histogram`
- Field: `@timestamp`
- Interval: `3h`
- Custom Label: `Time (3h)`

**Split series**
- Aggregation: `Terms`
- Field: `data.win.system.eventID`
- Size: `5`
- Order: `Descending`
- Custom Label: `Event ID`

Sub Aggregation: ❌ None

---

## Panel 5 — SOC | RDP | Internal | IP → User → Host

**Purpose:** Internal subnet RDP investigation view: source IP → user → destination host.

**DQL**
```dql
data.win.system.eventID:4624 AND data.win.eventdata.logonType:10 AND data.win.eventdata.ipAddress:(10.38.1.* OR 172.16.16.*) AND NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Data Table  
**Metric**
- Aggregation: `Count`
- Custom Label: `Internal RDP Success`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `data.win.eventdata.ipAddress`
   - Size: `50`
   - Order: `Descending`
   - Custom Label: `Source IP`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Size: `50`
   - Order: `Descending`
   - Custom Label: `User`

3) Sub Aggregation: `Terms`
   - Field: `agent.name`
   - Size: `50`
   - Order: `Descending`
   - Custom Label: `Host`

---

## Panel 6 — SOC | RDP | Success | SSL VPN Users (IP → User → Host)

**Purpose:** RDP success originating from SSL VPN IP pool (IPv4 pool + IPv6 prefix).

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
**Metric**
- Aggregation: `Count`
- Custom Label: `RDP Success`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `data.win.eventdata.ipAddress`
   - Size: `50`
   - Order: `Descending`
   - Custom Label: `VPN Source IP`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Size: `50`
   - Order: `Descending`
   - Custom Label: `User`

3) Sub Aggregation: `Terms`
   - Field: `agent.name`
   - Size: `50`
   - Order: `Descending`
   - Custom Label: `Host`

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
**Metric**
- Aggregation: `Unique Count`
- Field: `agent.name`
- Custom Label: `Distinct Target Hosts`

**Bucket**
- Aggregation: `Terms`
- Field: `data.win.eventdata.ipAddress`
- Size: `20`
- Order by: `Distinct Target Hosts`
- Order: `Descending`
- Custom Label: `VPN Source IP`

Sub Aggregation: ❌ None

---

## Panel 8 — SOC | RDP | Success | Public IP (Tripwire)

**Purpose:** Tripwire for direct public-origin RDP success (expected **0** in hardened environments).

**DQL**
```dql
data.win.system.eventID:4624
AND data.win.eventdata.logonType:10
AND data.win.eventdata.ipAddress:*
AND NOT data.win.eventdata.targetUserName:*$
AND NOT data.win.eventdata.ipAddress:(10.* OR 172.16.* OR 192.168.*)
AND NOT data.win.eventdata.ipAddress:("fe80*" OR "fd*" OR "fc*" OR "2001*" OR "::1")
```

**Visualization:** Horizontal Bar  
**Metric**
- Aggregation: `Count`
- Custom Label: `RDP Success`

**Bucket**
- Aggregation: `Terms`
- Field: `data.win.eventdata.ipAddress`
- Size: `20`
- Order: `Descending`
- Custom Label: `Public Source IP`

Sub Aggregation: ❌ None

---

END OF DASHBOARD-2 TEMPLATE
