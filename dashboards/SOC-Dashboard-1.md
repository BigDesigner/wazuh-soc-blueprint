# SOC Dashboard-1 — Authentication & Correlation (Production Template)

**Index pattern:** `wazuh-alerts-*`
**Time range:** `Last 24 hours` (default; override per investigation)
**Query language:** **DQL**

> Dashboard-1 purpose: authentication failure/success baselining + fast correlation to source IP and targeted users.

---

## Panel 1 — SOC | KPI | Failed Logins

**Purpose:** Global authentication failure indicator.

**DQL**
```dql
rule.groups:authentication_failed
```

**Visualization:** Metric

**Metrics**
- Aggregation: `Count`
- Custom Label: `Failed Logins`

**SOC notes**
- Sudden jump → brute-force / password spray.
- Correlate with **Panel 2** (Source IP) and **Panel 3** (Target users).
- Severity guidance: **Medium → High** if spike detected.

---

## Panel 2 — SOC | Top | Source IP (Failed Logins)

**Purpose:** Top attacking source IP addresses (user accounts only).

**DQL**
```dql
rule.groups:authentication_failed
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

**SOC notes**
- Public IP → external brute-force.
- Internal IP → lateral movement risk.
- Action: reputation check + firewall review.
- Severity: **High** if repeated.

---

## Panel 3 — SOC | Top | Target Users (Failed Logins)

**Purpose:** Most targeted user accounts (exclude machine accounts).

**DQL**
```dql
rule.groups:authentication_failed
AND data.win.eventdata.targetUserName:*
AND NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Vertical Bar

**Metrics**
- Aggregation: `Count`
- Custom Label: `Failed Attempts`

**Buckets**
- Aggregation: `Terms`
- Field: `data.win.eventdata.targetUserName`
- Order by: `Count`
- Order: `Descending`
- Size: `15`
- Custom Label: `Target User`

**SOC notes**
- Admin accounts targeted → privilege escalation attempt.
- Single user heavy targeting → password guessing/spray.
- Severity: **Medium → High** (High if privileged users).

---

## Panel 4 — SOC | Spike | Failed Logins (5m)

**Purpose:** Detect brute-force timing pattern.

**DQL**
```dql
rule.groups:authentication_failed
```

**Visualization:** Line Chart

**Metrics**
- Aggregation: `Count`
- Custom Label: `Events`

**Buckets**
- X-axis:
  - Aggregation: `Date Histogram`
  - Field: `@timestamp`
  - Interval: `5m`
  - Custom Label: `Time (5m)`

**SOC notes**
- Detection logic (baseline starter): **>50 events / 5 minutes** → probable brute wave.
- Flat continuous → credential stuffing.
- Sharp spike → password spray burst.
- Severity: **High**.

---

## Panel 5 — SOC | Source IP Spike (5m > 15 fails)

**Purpose:** Identify concentrated attack sources (per-IP, short window).

**DQL**
```dql
rule.groups:authentication_failed
AND data.win.eventdata.ipAddress:*
AND NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Bar Chart (Split by IP + timeline)

**Metrics**
- Aggregation: `Count`
- Custom Label: `Failed Attempts`

**Buckets**
- Split series:
  - Aggregation: `Terms`
  - Field: `data.win.eventdata.ipAddress`
  - Order by: `Count`
  - Order: `Descending`
  - Size: `10`
  - Custom Label: `Source IP`
- X-axis:
  - Aggregation: `Date Histogram`
  - Field: `@timestamp`
  - Interval: `5m`
  - Custom Label: `Time (5m)`

**SOC notes**
- Operational threshold: manual review if **any IP > 15 failures / 5 minutes**.
- Severity: **High**; **Critical** if external IP + admin targeting.

---

## Panel 6 — SOC | Correlation | Failed vs Success (Users)

**Purpose:** User-based authentication behavior correlation (failed vs success).

**DQL**
```dql
rule.groups:(authentication_failed OR authentication_success)
AND data.win.eventdata.targetUserName:*
AND NOT data.win.eventdata.targetUserName:*$
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Events`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Order by: `Count`
   - Order: `Descending`
   - Size: `20`
   - Custom Label: `Target User`
2) Aggregation: `Terms`
   - Field: `rule.groups`
   - Order by: `Count`
   - Order: `Descending`
   - Size: `10`
   - Custom Label: `Auth Result`

**SOC notes**
- Many fails + later success → possible credential compromise.
- Success only → normal behavior.
- Fail only → brute attempt.
- Severity: **Medium → High** (escalate on fail→success pattern).

---

## Panel 7 — SOC | Network Logons | Success (Users Only)

**Purpose:** Successful SMB/network logons (LogonType 3) for user accounts.

**DQL**
```dql
rule.groups:authentication_success
AND data.win.eventdata.logonType:3
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

**SOC notes**
- Sudden rise → lateral movement.
- Service accounts dominating → may be expected.
- User accounts spiking → suspicious workstation pivot.
- Severity: **Medium**; **High** if unusual pivot patterns.

---

END OF DASHBOARD-1 TEMPLATE
