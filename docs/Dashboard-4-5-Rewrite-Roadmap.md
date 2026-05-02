# 🗺️ AI Agent — Dashboard 4-5 Yeniden Yazım Yol Haritası

**Versiyon:** 1.0  
**Bağımlılık:** Bu dokümanı uygulamadan önce `AI-Agent-Panel-Standard.md` ve `AI-Agent-Overlap-Rules.md` okunmuş olmalıdır.

---

## 📌 Amaç

Dashboard 4 (Persistence & Lateral Movement) ve Dashboard 5 (Threat Intel & IOC) dosyalarını, Dashboard 1-2-3 ile aynı mimari standarta yükseltmek.

---

## 🔴 Mevcut Durum — D4 Sorunları

| # | Sorun | Detay |
|---|-------|-------|
| 1 | Header formatı yanlış | `# SOC Dashboard-4 — ... (Production Template)` formatı yok |
| 2 | `KQL:` etiketi kullanılmış | 5 panelde de `KQL:` yazıyor, `**DQL**` olmalı |
| 3 | Panel başlıkları `#` (h1) | `## Panel N — SOC \| ...` formatı gerekli |
| 4 | Purpose formatı yanlış | `Purpose:\nTask-based persistence.` → `**Purpose:** ...` olmalı |
| 5 | Metrics bölümü eksik | Hiçbir panelde `**Metrics**` bölümü yok |
| 6 | Buckets bölümü eksik | P1, P2, P4, P5'te bucket tanımı yok |
| 7 | SOC notes bölümü eksik | Hiçbir panelde SOC notes yok |
| 8 | Severity formatı yanlış | `Severity:\nHigh` → SOC notes içinde `**High**` olmalı |
| 9 | Code fence dili yok | DQL sorguları code fence içinde değil |
| 10 | `Columns:` formatı standart dışı | Bucket tanımına dönüştürülmeli + isteğe bağlı Columns eklenebilir |

---

## 🔴 Mevcut Durum — D5 Sorunları

| # | Sorun | Detay |
|---|-------|-------|
| 1 | Header formatı yanlış | D4 ile aynı sorun |
| 2 | `KQL:` etiketi kullanılmış | 5 panelde de `KQL:` yazıyor |
| 3 | Panel başlıkları `#` (h1) | D4 ile aynı sorun |
| 4 | Purpose formatı yanlış | D4 ile aynı sorun |
| 5 | Metrics bölümü eksik | Hiçbir panelde yok |
| 6 | Buckets bölümü eksik/kısmi | P2 hariç bucket detayı yok |
| 7 | SOC notes bölümü eksik | Hiçbir panelde yok |
| 8 | Severity formatı yanlış | D4 ile aynı sorun |
| 9 | `Buckets:` kısa formatı | `Y: data.destip` → tam format gerekli |

---

## 📋 Dashboard-4 Yeniden Yazım Planı

### D4 Yeni Header

```markdown
# SOC Dashboard-4 — Persistence & Lateral Movement (Production Template)

**Index pattern:** `wazuh-alerts-*`  
**Time range:** `Last 24 hours` (default; override per investigation)  
**Query language:** **DQL**

> Dashboard-4 purpose: Detect persistence mechanisms (scheduled tasks, registry autorun, service creation) and lateral movement indicators (SMB spikes, NTLM relay/PtH).

---
```

### D4 Panel Planı (5 Panel)

---

#### Panel 1 — SOC | Persistence | Scheduled Tasks (4698)

**Mevcut sorunlar:** Bucket yok, Metrics yok, SOC notes yok  
**Yeniden yazımda:**

- DQL: `data.win.system.eventID:4698` → code fence + `**DQL**` etiketi
- Visualization: Data Table
- Metrics: Count → `Scheduled Tasks Created`
- Bucket 1: `agent.name` → Host
- Bucket 2: `data.win.eventdata.taskName` (veya `data.win.eventdata.subjectUserName`) → Task Creator / Task Name
- SOC notes: Yeni task creation on unexpected host → investigate. Off-hours task creation → suspicious. Correlate with 4672/4688. Severity: **High**

---

#### Panel 2 — SOC | Persistence | Registry Run/RunOnce Keys

**Mevcut sorunlar:** Bucket yok, Metrics yok, SOC notes yok  
**Yeniden yazımda:**

- DQL: `rule.groups:registry AND data.win.eventdata.targetObject:(*\\Run\\* OR *\\RunOnce\\*)` → code fence
- Visualization: Data Table
- Metrics: Count → `Registry Modifications`
- Bucket 1: `agent.name` → Host
- Bucket 2: `data.win.eventdata.targetObject` → Registry Key
- SOC notes: New Run/RunOnce entry → potential malware autostart. Correlate with process creation (4688/Sysmon). Unknown binary paths → investigate immediately. Severity: **High**

---

#### Panel 3 — SOC | Lateral | SMB Logon Spike (Type 3, 5m)

**Mevcut sorunlar:** Bucket'ta sadece `Date Histogram @timestamp (5m)` notu, tam format yok  
**Overlap notu:** D1-P7 ile aynı DQL ama farklı viz (Line vs H.Bar) → **Tamamlayıcı**

**Yeniden yazımda:**

- DQL: `rule.groups:authentication_success AND data.win.eventdata.logonType:3 AND data.win.eventdata.targetUserName:* AND NOT data.win.eventdata.targetUserName:*$`
- Visualization: Line Chart
- Metrics: Count → `SMB Logon Events`
- Bucket X-axis: Date Histogram → @timestamp → Interval 5m
- SOC notes: Sudden spike → lateral movement burst. Flat elevated baseline → possible worm propagation. Correlate with Dashboard-1 Panel 7 (user-level SMB baseline). Severity: **High**

---

#### Panel 4 — SOC | Lateral | NTLM Logons (Relay / PtH Context)

**Mevcut sorunlar:** Bucket yok (sadece Columns listesi), Metrics yok, SOC notes yetersiz  
**Yeniden yazımda:**

- DQL: `data.win.system.eventID:4624 AND data.win.eventdata.authenticationPackageName:NTLM AND data.win.eventdata.targetUserName:* AND NOT data.win.eventdata.targetUserName:*$`
- Visualization: Data Table
- Metrics: Count → `NTLM Logon Events`
- Bucket 1: `agent.name` → Host
- Bucket 2: `data.win.eventdata.targetUserName` → User
- Bucket 3: `data.win.eventdata.ipAddress` → Source IP
- Columns (display context): @timestamp, agent.name, targetUserName, ipAddress, workstationName
- SOC notes: NTLM in Kerberos-only environment → suspicious relay attempt. Same IP authenticating to multiple hosts → Pass-the-Hash indicator. Unexpected NTLM from server segment → investigate. Severity: **Medium → High** (context dependent)

---

#### Panel 5 — SOC | Persistence | Service Creation (7045)

**Mevcut sorunlar:** Bucket yok (sadece Columns listesi), Metrics yok, SOC notes yok  
**Yeniden yazımda:**

- DQL: `data.win.system.eventID:7045`
- Visualization: Data Table
- Metrics: Count → `Services Created`
- Bucket 1: `agent.name` → Host
- Bucket 2: `data.win.eventdata.serviceName` → Service Name
- Columns (display context): @timestamp, agent.name, serviceName, imagePath, accountName
- SOC notes: Unknown service binary path → possible backdoor. Service running as SYSTEM → high-risk persistence. Service from temp/download folders → immediate investigation. Correlate with 4697 (Service Installed event). Severity: **Critical**

---

## 📋 Dashboard-5 Yeniden Yazım Planı

### D5 Yeni Header

```markdown
# SOC Dashboard-5 — Threat Intelligence & IOC Monitoring (Production Template)

**Index pattern:** `wazuh-alerts-*`  
**Time range:** `Last 24 hours` (default; override per investigation)  
**Query language:** **DQL**

> Dashboard-5 purpose: Surface threat intelligence matches, suspicious outbound destinations, malware hash detections, DNS anomalies, and MITRE ATT&CK technique distribution.

---
```

### D5 Panel Planı (5 Panel)

---

#### Panel 1 — SOC | TI | Threat Intel Matches

**Mevcut sorunlar:** Bucket yok (sadece Columns), Metrics yok, SOC notes yetersiz  
**Yeniden yazımda:**

- DQL: `rule.groups:threat_intel`
- Visualization: Data Table
- Metrics: Count → `Threat Intel Hits`
- Bucket 1: `agent.name` → Host
- Bucket 2: `rule.description` → Detection Rule
- Columns (display context): @timestamp, agent.name, data.srcip, data.destip, rule.description, rule.level
- SOC notes: Any hit → validate IOC freshness and asset criticality. Multiple hits from same host → possible active compromise. High rule.level (≥12) → immediate escalation. Correlate with Dashboard-2 (RDP) and Dashboard-4 (Persistence) for lateral indicators. Severity: **High → Critical** (depends on rule.level and asset criticality)

---

#### Panel 2 — SOC | TI | Suspicious Outbound Destinations (Non-RFC1918)

**Mevcut sorunlar:** Bucket kısa format (`Y: data.destip`), Metrics yok, SOC notes yok  
**Yeniden yazımda:**

- DQL: `rule.groups:firewall AND NOT data.destip:(10.* OR 192.168.* OR 172.16.*)`
- Visualization: Horizontal Bar
- Metrics: Count → `Outbound Connections`
- Bucket: Terms → data.destip → Size 15 → `Destination IP`
- SOC notes: High-volume destination → potential C2 beaconing. Destinations in sanctioned countries → geopolitical risk. Correlate destination IP with threat intel feeds. New/unseen destination → hunting pivot. Severity: **Medium → High** (escalate on TI match or anomalous volume)

---

#### Panel 3 — SOC | TI | Malware Hash Detections

**Mevcut sorunlar:** Bucket yok (sadece Columns), Metrics yok, SOC notes minimum  
**Yeniden yazımda:**

- DQL: `rule.groups:malware`
- Visualization: Data Table
- Metrics: Count → `Malware Detections`
- Bucket 1: `agent.name` → Host
- Bucket 2: `rule.description` → Detection
- Columns (display context): @timestamp, agent.name, file.hash, file.path, rule.description
- SOC notes: Any detection → immediate host isolation assessment. Multiple hosts with same hash → active propagation. File in system32/temp/download → high-risk path. Validate hash on VirusTotal/MISP. Severity: **Critical**

---

#### Panel 4 — SOC | TI | DNS Suspicious TLD Watchlist

**Mevcut sorunlar:** Bucket yok (sadece Columns), Metrics yok, SOC notes yok  
**Yeniden yazımda:**

- DQL: `rule.groups:dns AND data.query:(*.xyz OR *.top OR *.ru OR *.click OR *.gq)`
- Visualization: Data Table
- Metrics: Count → `Suspicious DNS Queries`
- Bucket 1: `agent.name` → Host
- Bucket 2: `data.query` → Domain
- Bucket 3: `data.srcip` → Source IP
- SOC notes: Repeated queries to same suspicious domain → possible C2 callback. High query volume from single host → DNS tunneling indicator. New domain not in baseline → hunting pivot. Add confirmed malicious domains to blocklist. Severity: **Medium → High** (context dependent)

---

#### Panel 5 — SOC | TI | MITRE ATT&CK Technique Distribution

**Mevcut sorunlar:** Bucket kısa format, Metrics yok, SOC notes yok  
**Yeniden yazımda:**

- DQL: `rule.mitre.id:*`
- Visualization: Pie
- Metrics: Count → `Technique Hits`
- Bucket: Terms → rule.mitre.id → Size 15 → `MITRE Technique`
- SOC notes: Dominant technique → indicates active attack phase. Unexpected technique appearance → new threat vector. Use for weekly/monthly SOC reporting and coverage gap analysis. Correlate with MITRE Coverage Matrix for detection completeness. Severity: **Low** (informational / reporting)

---

## ⚡ Yeniden Yazım Sırası

| Adım | İş | Dosya |
|------|-----|-------|
| 1 | Dashboard-4 header yaz | `SOC-Dashboard-4-Persistence.md` |
| 2 | D4 Panel 1-5 sırayla yaz | Aynı dosya |
| 3 | D4 footer ekle | Aynı dosya |
| 4 | Dashboard-5 header yaz | `SOC-Dashboard-5-Threat-Intel-IOC.md` |
| 5 | D5 Panel 1-5 sırayla yaz | Aynı dosya |
| 6 | D5 footer ekle | Aynı dosya |
| 7 | Overlap kontrolü | Tüm dashboard'lar arası final kontrol |

---

## ✅ Kalite Kontrol Listesi

Yeniden yazım tamamlandıktan sonra her panel için kontrol:

```
□ Panel başlığı: ## Panel N — SOC | Kategori | İsim
□ Purpose: **Purpose:** ile başlıyor, tek cümle
□ DQL: **DQL** etiketi, ```dql code fence, KQL etiketi YOK
□ Visualization: İzin verilen tiplerden biri
□ Metrics: Aggregation + Custom Label (Field gerekiyorsa var)
□ Buckets: Tam format (Field, Order by, Order, Size, Custom Label)
□ SOC notes: En az 2 analiz notu + → operatörü + Severity
□ Ayırıcı: --- ile biten
□ Overlap: Mevcut panellerle çakışma kontrolü yapıldı
□ Cross-reference: İlişkili panellere atıf SOC notes'a eklendi
```
