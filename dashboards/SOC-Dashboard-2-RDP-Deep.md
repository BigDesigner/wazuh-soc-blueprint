# SOC Dashboard 2 – VPN & RDP Behavioral Monitoring

Index Pattern: `wazuh-alerts-*`  
Time Range (recommended): `Last 7 days`  
Query Language: **DQL**

---

# PANEL 1
## SOC | RDP Success Top Users

### Purpose
RDP (LogonType 10) ile başarılı oturum açan kullanıcıları gösterir.  
Operasyonel baseline ve anormal kullanıcı yoğunluğu tespiti için kullanılır.

### DQL
```dql
data.win.system.eventID:4624
AND data.win.eventdata.logonType:10
AND NOT data.win.eventdata.targetUserName:*$
```

### Visualization
Type: **Horizontal Bar**

### Metrics
- Aggregation: `Count`
- Custom Label: `Successful Logons`

### Buckets
- Aggregation: `Terms`
- Field: `data.win.eventdata.targetUserName`
- Order by: `Metric: Successful Logons`
- Order: `Descending`
- Size: `10`
- Custom Label: `User Account`

Sub Aggregation: ❌ None

---

# PANEL 2
## SOC | RDP Failed Top Source IP

### Purpose
RDP brute-force veya parola denemesi yapan kaynak IP’leri gösterir.

### DQL
```dql
data.win.system.eventID:4625
AND data.win.eventdata.logonType:10
AND data.win.eventdata.ipAddress:*
AND NOT data.win.eventdata.targetUserName:*$
```

### Visualization
Type: **Horizontal Bar**

### Metrics
- Aggregation: `Count`
- Custom Label: `Failed Attempts`

### Buckets
- Aggregation: `Terms`
- Field: `data.win.eventdata.ipAddress`
- Order by: `Metric: Failed Attempts`
- Order: `Descending`
- Size: `10`
- Custom Label: `Source IP`

Sub Aggregation: ❌ None

---

# PANEL 3
## SOC | Authentication | Logon Type Distribution

### Purpose
LogonType dağılımını gösterir (Interactive, Network, RDP vs).  
Beklenmeyen oturum tiplerini analiz etmek için kullanılır.

### DQL
```dql
data.win.system.eventID:(4624 OR 4625)
AND NOT data.win.eventdata.targetUserName:*$
```

### Visualization
Type: **Data Table**

### Metrics
- Aggregation: `Count`
- Custom Label: `Attempts`

### Buckets
1) Aggregation: `Terms`
   - Field: `data.win.eventdata.ipAddress`
   - Size: `20`
   - Custom Label: `Source IP`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Size: `20`
   - Custom Label: `Target User`

3) Sub Aggregation: `Terms`
   - Field: `data.win.system.eventID`
   - Size: `5`
   - Custom Label: `Event ID`

---

# PANEL 4
## SOC | SSL VPN | Success vs Failed (Windows Auth)

### Purpose
VPN IP havuzundan gelen Windows authentication success vs fail oranlarını gösterir.  
VPN brute-force veya credential abuse tespiti için kullanılır.

### DQL
```dql
data.win.system.eventID:(4624 OR 4625)
AND data.win.eventdata.ipAddress:*
AND NOT data.win.eventdata.targetUserName:*$
AND (
  data.win.eventdata.ipAddress:(10.212.134.200 OR 10.212.134.201 OR 10.212.134.202 OR 10.212.134.203 OR 10.212.134.204 OR 10.212.134.205 OR 10.212.134.206 OR 10.212.134.207 OR 10.212.134.208 OR 10.212.134.209 OR 10.212.134.210)
  OR data.win.eventdata.ipAddress:"fdff:ffff*"
)
```

### Visualization
Type: **Vertical Bar**  
Mode: **Stacked**

### Metrics
- Aggregation: `Count`
- Custom Label: `VPN Auth Events`

### X-Axis
- Aggregation: `Date Histogram`
- Field: `@timestamp`
- Interval: `5m`
- Custom Label: `Time (5m)`

### Split Series
- Aggregation: `Terms`
- Field: `data.win.system.eventID`
- Size: `2`
- Order by: `Metric: VPN Auth Events`
- Custom Label: `EventID (4624=Success / 4625=Fail)`

Sub Aggregation: ❌ None

---

# PANEL 5
## SOC | RDP | Success | Internal LAN Users

### Purpose
İç ağdan yapılan RDP bağlantılarını izler.  
Normal operasyonel davranışı ayırt etmek için kullanılır.

### DQL
```dql
data.win.system.eventID:4624
AND data.win.eventdata.logonType:10
AND data.win.eventdata.ipAddress:(10.38.1.* OR 172.16.16.*)
AND NOT data.win.eventdata.targetUserName:*$
```

### Visualization
Type: **Data Table**

### Metrics
- Aggregation: `Count`
- Custom Label: `RDP Success`

### Buckets
1) Aggregation: `Terms`
   - Field: `data.win.eventdata.ipAddress`
   - Custom Label: `Source IP`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Custom Label: `User`

3) Sub Aggregation: `Terms`
   - Field: `agent.name`
   - Custom Label: `Target Host`

---

# PANEL 6
## SOC | RDP | Admin Successful Logons

### Purpose
Administrator veya admin pattern içeren hesapların başarılı RDP loginlerini izler.  
Privileged account misuse tespiti için kullanılır.

### DQL
```dql
data.win.system.eventID:4624
AND data.win.eventdata.logonType:10
AND NOT data.win.eventdata.targetUserName:*$
AND (
  data.win.eventdata.targetUserName:administrator
  OR data.win.eventdata.targetUserName:*admin*
)
```

### Visualization
Type: **Data Table**

### Metrics
- Aggregation: `Count`
- Custom Label: `Admin RDP Success`

### Buckets
1) Aggregation: `Terms`
   - Field: `data.win.eventdata.targetUserName`
   - Custom Label: `Admin Account`

2) Sub Aggregation: `Terms`
   - Field: `agent.name`
   - Custom Label: `Target Host`

---

# PANEL 7
## SOC | SSL VPN | Multi-Host Access Count

### Purpose
Aynı VPN IP’nin kaç farklı host’a login olduğunu gösterir.  
VPN sonrası lateral movement erken tespiti için kullanılır.

### DQL
```dql
data.win.system.eventID:4624
AND data.win.eventdata.ipAddress:*
AND NOT data.win.eventdata.targetUserName:*$
AND (
  data.win.eventdata.ipAddress:(10.212.134.200 OR 10.212.134.201 OR 10.212.134.202 OR 10.212.134.203 OR 10.212.134.204 OR 10.212.134.205 OR 10.212.134.206 OR 10.212.134.207 OR 10.212.134.208 OR 10.212.134.209 OR 10.212.134.210)
  OR data.win.eventdata.ipAddress:"fdff:ffff*"
)
```

### Visualization
Type: **Data Table**

### Metrics
- Aggregation: `Unique Count`
- Field: `agent.name`
- Custom Label: `Distinct Target Hosts`

### Buckets
- Aggregation: `Terms`
- Field: `data.win.eventdata.ipAddress`
- Order by: `Metric: Distinct Target Hosts`
- Order: `Descending`
- Size: `20`
- Custom Label: `VPN Source IP`

Sub Aggregation: ❌ None

---

# Dashboard Strategy Summary

Dashboard 2 focuses on:
- RDP success/failure behavior
- VPN authentication monitoring (via VPN IP pool)
- Privileged usage control
- Internal vs VPN segmentation
- Early lateral movement detection

---

END OF DASHBOARD-2 TEMPLATE
