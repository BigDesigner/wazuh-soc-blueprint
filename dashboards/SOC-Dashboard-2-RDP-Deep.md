# SOC Dashboard-2 — Deep RDP Monitoring (Production Template)

**Index pattern:** `wazuh-alerts-*`  
**Time range:** `Last 24 hours` (default; override per investigation)  
**Query language:** **DQL**

> Dashboard-2 purpose: Comprehensive RDP behavioral monitoring, internal movement tracking, SSL VPN IP correlation, and public exposure tripwires.

---

## Panel 1 — SOC | RDP | Success | Top Users

**Purpose:** RDP (LogonType 10) successful logons by user (baseline tracking and anomaly detection).

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
- Size: 15
- Custom Label: `User`

**SOC notes**
- Regular IT administrators should dominate this baseline.
- Standard business users logging in via RDP (LogonType 10) → possible stolen credentials or unapproved remote tool usage.
- Correlate with Panel 5 (Internal IP→User→Host) and Dashboard-1 Panel 6 (Failed vs Success).
- Severity: **Medium** (High if non-admin or off-hours).

---

## Panel 2 — SOC | RDP | Failed | Top Source IP

**Purpose:** Failed RDP logon attempts grouped by source IP address (brute-force and password guessing).

**DQL**
```dql
data.win.system.eventID:4625
AND data.win.eventdata.logonType:10
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
- Size: 15
- Custom Label: `Source IP`

**SOC notes**
- High volume from a single internal IP → internal lateral movement brute-force attempt.
- Correlate with Dashboard-1 Panel 2 (Global failed source IPs across all logon types).
- Operational threshold: >15 failed attempts in 5 minutes from any source IP → investigate source host for compromise.
- Severity: **High**

---

## Panel 3 — SOC | Authentication | Logon Type Distribution

**Purpose:** Combined drill-down of RDP success (4624 + LogonType 10) and authentication failures (4625), excluding loopback and IPv6 noise.

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
   - Order by: `Count`
   - Order: `Descending`
   - Size: 20
   - Custom Label: `Source IP`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 20
   - Custom Label: `Target User`

3) Sub Aggregation: `Terms`
   - Field: `data.win.system.eventID`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Event ID`

**SOC notes**
- High volume of 4625 followed by a single 4624 for the same User and IP → confirmed credential compromise.
- Multiple target users from a single source IP → password spray targeting RDP endpoints.
- Severity: **Medium → High** (escalate immediately upon fail→success transition).

---

## Panel 4 — SOC | RDP | Internal | Timeline (4624/4625)

**Purpose:** Internal subnet RDP activity timeline tracking success vs failure trends.

**DQL**
```dql
(
  (data.win.system.eventID:4624 AND data.win.eventdata.logonType:10)
  OR
  (data.win.system.eventID:4625)
)
AND data.win.eventdata.ipAddress:(10.38.1.* OR 172.16.16.*)
AND NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Vertical Bar  
Mode: **Stacked**

**Metrics**
- Aggregation: `Count`
- Custom Label: `Internal RDP Events`

**Buckets**
- X-axis:
  - Aggregation: `Date Histogram`
  - Field: `@timestamp`
  - Interval: `3h`
  - Custom Label: `Time (3h)`
- Split series:
  - Aggregation: `Terms`
  - Field: `data.win.system.eventID`
  - Order by: `Count`
  - Order: `Descending`
  - Size: 5
  - Custom Label: `Event ID`

**SOC notes**
- Failures (4625) dominating baseline indicates automated internal scanning or brute-force wave.
- Spikes in successful logons (4624) outside normal maintenance windows require correlation with change tickets.
- Severity: **Medium** (High if failure bursts detected).

---

## Panel 5 — SOC | RDP | Internal | IP → User → Host

**Purpose:** Internal subnet RDP investigation view mapping source IP to user to target destination host.

**DQL**
```dql
data.win.system.eventID:4624
AND data.win.eventdata.logonType:10
AND data.win.eventdata.ipAddress:(10.38.1.* OR 172.16.16.*)
AND NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Internal RDP Success`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `data.win.eventdata.ipAddress`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 50
   - Custom Label: `Source IP`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 50
   - Custom Label: `User`

3) Sub Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 50
   - Custom Label: `Target Host`

**SOC notes**
- Maps workstation-to-server and workstation-to-workstation lateral pivot pathways.
- Workstation connecting to multiple peer workstations via RDP → anomalous pivot behavior.
- Correlate with Sysmon Event ID 1 (Process Create) on the source IP host.
- Severity: **High** if unexpected source workstation.

---

## Panel 6 — SOC | RDP | Success | SSL VPN Users (IP → User → Host)

**Purpose:** Monitor RDP successful connections originating from authorized SSL VPN IP pools.

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
   - Order by: `Count`
   - Order: `Descending`
   - Size: 50
   - Custom Label: `VPN Source IP`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 50
   - Custom Label: `User`

3) Sub Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 50
   - Custom Label: `Target Host`

**SOC notes**
- Validates legitimate remote worker access into server infrastructure.
- Unfamiliar user account logging in from VPN pool → potential compromised VPN account.
- Correlate with VPN gateway authentication and multi-factor authentication (MFA) logs.
- Severity: **Medium** (High if privileged accounts connect outside normal schedule).

---

## Panel 7 — SOC | SSL VPN | Multi-Host Access Count

**Purpose:** Detect potential lateral movement from VPN tunnels by identifying single VPN IPs accessing multiple destination hosts.

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
- Aggregation: `Unique Count`
- Field: `agent.name`
- Custom Label: `Distinct Target Hosts`

**Buckets**
- Aggregation: `Terms`
- Field: `data.win.eventdata.ipAddress`
- Order by: `Distinct Target Hosts`
- Order: `Descending`
- Size: 20
- Custom Label: `VPN Source IP`

**SOC notes**
- Single VPN IP accessing 3+ distinct hosts within 24 hours → strong lateral movement indicator.
- Standard remote employees typically connect to a single designated workstation.
- Action: Contact user to verify active workload; escalate if unconfirmed.
- Severity: **High → Critical**

---

## Panel 8 — SOC | RDP | Success | Public IP (Tripwire)

**Purpose:** High-severity tripwire detecting direct successful RDP logons originating from public Internet IPs.

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

**Metrics**
- Aggregation: `Count`
- Custom Label: `RDP Success`

**Buckets**
- Aggregation: `Terms`
- Field: `data.win.eventdata.ipAddress`
- Order by: `Count`
- Order: `Descending`
- Size: 20
- Custom Label: `Public Source IP`

**SOC notes**
- Operational threshold: **Expected 0 events** in hardened environments.
- Any single event represents direct public Internet exposure of port 3389 without VPN or tunnel.
- Immediate action: Isolate host, terminate session, and check perimeter firewall NAT rules.
- Severity: **Critical**

---

END OF DASHBOARD-2 TEMPLATE
