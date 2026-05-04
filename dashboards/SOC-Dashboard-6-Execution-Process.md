# SOC Dashboard-6 — Execution & Process Monitoring (Production Template)

**Index pattern:** `wazuh-alerts-*`  
**Time range:** `Last 24 hours` (default; override per investigation)  
**Query language:** **DQL**

> Dashboard-6 purpose: Monitor for malicious code execution, suspicious process trees, credential dumping attempts, and script-based attacks (PowerShell/CMD).

---

## Panel 1 — SOC | Execution | PowerShell Script Block Logging (4104)

**Purpose:** Detect potentially malicious PowerShell scripts being executed in the environment.

**DQL**
```dql
data.win.system.eventID:4104 
AND data.win.eventdata.scriptBlockText:(*invoke-* OR *iex* OR *base64* OR *enc* OR *bypass*)
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Suspicious PowerShell Scripts`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.scriptBlockText`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 5
   - Custom Label: `Script Block Content`

**SOC notes**
- High volume of 4104 with obfuscation keywords → possible active exploitation or C2 beaconing.
- Correlate with 4688/Sysmon EID 1 for the parent process (e.g., winword.exe spawning powershell.exe).
- Severity: **High**

---

## Panel 2 — SOC | Execution | Process Creation (4688 / Sysmon 1)

**Purpose:** Monitor for suspicious process creation, focusing on common attacker tools or LOLBAS binaries.

**DQL**
```dql
(data.win.system.eventID:4688 OR data.win.system.eventID:1)
AND data.win.eventdata.parentImage:(*winword.exe OR *excel.exe OR *powershell.exe OR *cmd.exe)
AND data.win.eventdata.image:(*whoami.exe OR *net.exe OR *nltest.exe OR *vssadmin.exe)
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Suspicious Process Spawns`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.image`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Child Process`

**SOC notes**
- Office applications spawning system tools → high probability of phishing success.
- `vssadmin.exe` execution → common indicator of ransomware (shadow copy deletion).
- Severity: **Critical**

---

## Panel 3 — SOC | Credential Access | LSASS Access (Sysmon 10)

**Purpose:** Detect potential credential dumping attempts (e.g., Mimikatz, Procdump) by monitoring access to the LSASS process.

**DQL**
```dql
data.win.system.eventID:10 
AND data.win.eventdata.targetImage:*lsass.exe 
AND NOT data.win.eventdata.sourceImage:(*msmpeng.exe OR *svchost.exe OR *lsass.exe)
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `LSASS Access Events`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.sourceImage`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Source Image (Potential Dumper)`

**SOC notes**
- Unknown or non-system process accessing LSASS → high risk of credential theft.
- Correlate with 4688/Sysmon 1 for the source image's creation time and parent.
- Severity: **Critical**

---

## Panel 4 — SOC | Defense Evasion | Process Injection (Sysmon 8 / 25)

**Purpose:** Detect code injection into other processes, a common technique for defense evasion and persistence.

**DQL**
```dql
(data.win.system.eventID:8 OR data.win.system.eventID:25)
AND NOT data.win.eventdata.sourceImage:(*chrome.exe OR *msedge.exe)
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Injected Process Events`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.targetImage`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Target Process`

**SOC notes**
- Unusual source image (e.g., `cmd.exe`, `powershell.exe`) injecting into `explorer.exe` or `svchost.exe`.
- Correlate with Dashboard-6 Panel 1 and 2 for preceding execution indicators.
- Severity: **High → Critical**

---

## Panel 5 — SOC | Execution | Long Command Line Analysis

**Purpose:** Identify potential encoded payloads or complex commands that are often used in malicious scripts.

**DQL**
```dql
(data.win.system.eventID:4688 OR data.win.system.eventID:1)
AND data.win.eventdata.commandLine.length > 500
```

**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `Long Command Lines`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `agent.name`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Host`

2) Sub Aggregation: `Terms`
   - Field: `data.win.eventdata.commandLine`
   - Order by: `Count`
   - Order: `Descending`
   - Size: 10
   - Custom Label: `Command Line`

**SOC notes**
- Extremely long commands often contain Base64 encoded scripts or heavy obfuscation.
- De-obfuscate the command line to reveal the actual intent.
- Severity: **Medium → High**

---

END OF DASHBOARD-6 TEMPLATE
