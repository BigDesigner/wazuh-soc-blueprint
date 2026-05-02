# 🤖 AI Agent — SOC Dashboard Panel Yazım Standardı

**Versiyon:** 1.0  
**Kaynak:** Dashboard 1-2-3 (Production — çalışan referans)  
**Amaç:** Bu doküman, bir AI Agent'ın yeni SOC dashboard paneli yazarken **uyması zorunlu** olan kuralları tanımlar. Dashboard 4, 5 ve sonrası bu standartla yeniden yazılacaktır.

---

## 📌 KURAL 0 — Genel İlkeler

1. Query dili **yalnızca DQL** olacaktır. `KQL` etiketi **yasaktır**. Wazuh Indexer (OpenSearch) ortamında KQL çalışmaz, yalnızca DQL çalışır.
2. Her panel **tek bir detection amacına** hizmet etmelidir. Bir panelde birden fazla ilişkisiz detection mantığı olmamalıdır.
3. Her panel bir sonraki AI Agent tarafından **copy-paste** ile OpenSearch Dashboards'a aktarılabilir detayda olmalıdır.
4. Hiçbir panel, mevcut bir başka panelin **aynı DQL + aynı visualization + aynı bucket** kombinasyonunu tekrarlamamalıdır (bkz. Overlap Kuralları).

---

## 📐 KURAL 1 — Dashboard Dosya Başlığı (Header)

Her dashboard dosyası şu header ile başlamalıdır:

```markdown
# SOC Dashboard-{N} — {Başlık} (Production Template)

**Index pattern:** `wazuh-alerts-*`  
**Time range:** `Last 24 hours` (default; override per investigation)  
**Query language:** **DQL**

> Dashboard-{N} purpose: {Tek cümlelik genel amaç açıklaması}.

---
```

### Header Kuralları

| Alan | Zorunlu mu? | Format | Açıklama |
|------|-------------|--------|----------|
| `# SOC Dashboard-{N}` | ✅ Evet | `# SOC Dashboard-{N} — {Başlık} (Production Template)` | N = dashboard numarası |
| Index pattern | ✅ Evet | `wazuh-alerts-*` | Sabit değer |
| Time range | ✅ Evet | `Last 24 hours` | Varsayılan, değişebilir notu ekle |
| Query language | ✅ Evet | `**DQL**` | Asla KQL yazmayın |
| Purpose (blockquote) | ✅ Evet | `> Dashboard-{N} purpose: ...` | Tek cümle, net amaç |

---

## 📐 KURAL 2 — Panel Yapısı (Zorunlu Sıralama)

Her panel aşağıdaki bölümleri, **bu sırayla** içermelidir:

```
## Panel {N} — SOC | {Kategori} | {İsim}

**Purpose:** {Açıklama}

**DQL**
```dql
{sorgu}
```

**Visualization:** {Tip}

**Metrics**
- Aggregation: `{Tip}`
- Field: `{alan}` ← (Count dışında zorunlu)
- Custom Label: `{etiket}`

**Buckets** veya **Buckets (Split rows order)**
{bucket tanımları}

**SOC notes**
- {analiz notu 1}
- {analiz notu 2}
- Severity: **{seviye}**

---
```

### Zorunlu Bölümler Kontrol Listesi

| # | Bölüm | Zorunlu | Atlanabilir Koşul |
|---|--------|---------|-------------------|
| 1 | `## Panel {N} — SOC \| ...` | ✅ Her zaman | — |
| 2 | `**Purpose:**` | ✅ Her zaman | — |
| 3 | `**DQL**` + code block | ✅ Her zaman | — |
| 4 | `**Visualization:**` | ✅ Her zaman | — |
| 5 | `**Metrics**` | ✅ Her zaman | — |
| 6 | `**Buckets**` | ⚠️ Koşullu | Yalnızca `Metric` (KPI) tipinde atlanabilir |
| 7 | `**SOC notes**` | ✅ Her zaman | — |
| 8 | Ayırıcı `---` | ✅ Her zaman | — |

---

## 📐 KURAL 3 — Panel Başlığı Formatı

```
## Panel {N} — SOC | {Kategori} | {İsim}
```

### Kurallar

- Heading seviyesi: `##` (h2) — **`#` (h1) kullanmayın**.
- `SOC |` prefix'i sabit.
- Kategori örnekleri: `KPI`, `Top`, `Spike`, `Correlation`, `RDP`, `Auth`, `Persistence`, `TI`
- İsim: Kısa, tanımlayıcı. Parantez içinde ek bağlam verilebilir.

### ✅ Doğru Örnekler
```
## Panel 1 — SOC | KPI | Failed Logins
## Panel 3 — SOC | Top | Target Users (Failed Logins)
## Panel 5 — SOC | Spike | Source IP (5m > 15 fails)
## Panel 6 — SOC | Correlation | Failed vs Success (Users)
## Panel 2 — SOC | Persistence | Scheduled Tasks (4698)
```

### ❌ Yanlış Örnekler
```
# 1) New Scheduled Tasks (4698)        ← h1 kullanmış, SOC | prefix yok
# Panel 2 — Admin Group Changes        ← SOC | prefix yok
## Service Creation                     ← Panel numarası yok
```

---

## 📐 KURAL 4 — Purpose Alanı

```
**Purpose:** {Tek cümle. Ne tespit edilecek, neyi izliyor.}
```

### Kurallar
- **Tek cümle** olmalı.
- İzleme hedefini net açıklamalı.
- Teknik terim kullanılabilir ama jargon olmadan.
- Parentez içinde Event ID veya LogonType belirtilebilir.

### ✅ Doğru Örnekler
```
**Purpose:** RDP (LogonType 10) successful logons by user (baseline + anomalies).
**Purpose:** Identify accounts added to privileged groups (potential privilege escalation or persistence).
**Purpose:** Tripwire for direct public-origin RDP success (expected **0** in hardened environments).
```

### ❌ Yanlış Örnekler
```
Purpose:
Task-based persistence.          ← Bold formatting yok, çok kısa, çok belirsiz

Purpose:
Autorun persistence.             ← Hangi mekanizma? Registry mi? Scheduled Task mi?
```

---

## 📐 KURAL 5 — DQL Bloku

```markdown
**DQL**
```dql
{sorgu}
```
```

### Kurallar

| Kural | Açıklama |
|-------|----------|
| Etiket `**DQL**` | `KQL:` veya `KQL` yazmak **yasaktır** |
| Code fence dili | ` ```dql ` kullanılmalı |
| Machine account filtresi | User-facing panellerde `AND NOT data.win.eventdata.targetUserName:*$` ekle |
| Loopback filtresi | IP tabanlı panellerde `AND NOT data.win.eventdata.ipAddress:("::1")` düşün |
| IPv6 filtresi | Gerekiyorsa `AND NOT data.win.eventdata.ipAddress:(fe80* OR 2001*)` ekle |
| Satır başı AND/OR | Uzun sorgularda okunabilirlik için her koşulu yeni satırda yaz |
| Wildcard kullanımı | Field varlık kontrolü için `fieldName:*` kullanılabilir |

### DQL Yapı Şablonları

**Basit (tek koşul):**
```dql
rule.groups:authentication_failed
```

**Filtrelenmiş (machine account + IP):**
```dql
rule.groups:authentication_failed
AND data.win.eventdata.ipAddress:*
AND NOT data.win.eventdata.targetUserName:*$
```

**EventID tabanlı:**
```dql
data.win.system.eventID:4672
AND data.win.eventdata.subjectUserName:*
AND NOT data.win.eventdata.subjectUserName:*$
```

**Çoklu EventID (OR):**
```dql
data.win.system.eventID:(4728 OR 4732 OR 4756)
```

**Karmaşık (gruplu OR + filtre):**
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

**Subnet / IP aralığı filtresi:**
```dql
AND NOT data.win.eventdata.ipAddress:(10.* OR 172.16.* OR 192.168.*)
```

---

## 📐 KURAL 6 — Visualization Tipi

```
**Visualization:** {Tip}
```

### İzin Verilen Tipler ve Kullanım Alanları

| Tip | Ne Zaman Kullanılır | Bucket Gerekli mi? |
|-----|---------------------|-------------------|
| `Metric` | KPI / tek sayı gösterimi | ❌ Hayır |
| `Horizontal Bar` | Top-N sıralaması (IP, user) | ✅ Evet |
| `Vertical Bar` | Top-N veya zaman bazlı karşılaştırma | ✅ Evet |
| `Line Chart` | Zaman serisi trend/spike | ✅ Evet (X-axis Date Histogram) |
| `Bar Chart` | Split series + timeline combo | ✅ Evet (Split + X-axis) |
| `Data Table` | Detaylı drill-down, çoklu boyut | ✅ Evet (Split rows) |
| `Pie` | Dağılım/oran gösterimi | ✅ Evet |

### Ek Görselleştirme Notları

- Vertical Bar + `Mode: **Stacked**` → ayrı belirtilmeli (bkz. D2-P4)
- `Line` yerine `Line Chart` yazılmalı
- `Bar Chart` tek başına kullanıldığında Split series + X-axis kombinasyonu demektir

---

## 📐 KURAL 7 — Metrics Bölümü

```markdown
**Metrics**
- Aggregation: `{Tip}`
- Field: `{alan}`          ← Count dışında zorunlu
- Custom Label: `{etiket}`
```

### İzin Verilen Aggregation Tipleri

| Aggregation | Field Gerekli mi? | Kullanım |
|-------------|-------------------|----------|
| `Count` | ❌ | Olay sayısı |
| `Unique Count` | ✅ | Benzersiz değer sayısı (ör. farklı host sayısı) |
| `Sum` | ✅ | Toplam |
| `Average` | ✅ | Ortalama |
| `Max` / `Min` | ✅ | Uç değer |

### ✅ Doğru Örnekler
```markdown
**Metrics**
- Aggregation: `Count`
- Custom Label: `Failed Logins`

**Metrics**
- Aggregation: `Unique Count`
- Field: `agent.name`
- Custom Label: `Distinct Target Hosts`
```

### ❌ Yanlış Örnekler
```
Metric:
Count                    ← Format yok, Custom Label yok
```

---

## 📐 KURAL 8 — Buckets Bölümü

Bucket yapısı, visualization tipine göre farklılık gösterir. Aşağıdaki varyasyonlar kabul edilir:

### Varyasyon A — Tekli Bucket (Bar / Pie)

```markdown
**Buckets**
- Aggregation: `Terms`
- Field: `{alan}`
- Order by: `Count`
- Order: `Descending`
- Size: `{N}`
- Custom Label: `{etiket}`
```

### Varyasyon B — Çoklu Bucket / Data Table (Split Rows)

```markdown
**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `{alan_1}`
   - Order by: `Count`
   - Order: `Descending`
   - Size: `{N}`
   - Custom Label: `{etiket_1}`

2) Sub Aggregation: `Terms`
   - Field: `{alan_2}`
   - Order by: `Count`
   - Order: `Descending`
   - Size: `{N}`
   - Custom Label: `{etiket_2}`
```

> **Not:** D3'te numaralama `1️⃣ Split Rows` formatında, D2'de `1)` formatında yapılmış. Standart olarak `1)` formatını kullanın. Emoji numaralama da kabul edilir.

### Varyasyon C — Timeline (X-axis + Split series)

```markdown
**Buckets**
- X-axis:
  - Aggregation: `Date Histogram`
  - Field: `@timestamp`
  - Interval: `{5m / 3h / 1h}`
  - Custom Label: `Time ({interval})`

- Split series:
  - Aggregation: `Terms`
  - Field: `{alan}`
  - Order by: `Count`
  - Order: `Descending`
  - Size: `{N}`
  - Custom Label: `{etiket}`
```

### Varyasyon D — Alt Aggregation Yoksa

```markdown
Sub Aggregation: ❌ None
```

Bu satır **isteğe bağlıdır**. Yalnızca netlik için eklenebilir (D2 stili).

### Bucket Parametreleri Referans

| Parametre | Zorunlu | Varsayılan | Açıklama |
|-----------|---------|------------|----------|
| `Aggregation` | ✅ | — | `Terms`, `Date Histogram` |
| `Field` | ✅ | — | OpenSearch field path |
| `Order by` | ✅ | `Count` | `Count` veya Custom Label adı |
| `Order` | ✅ | `Descending` | `Descending` / `Ascending` |
| `Size` | ✅ | — | Top-N sayısı |
| `Custom Label` | ✅ | — | Anlamlı etiket |
| `Interval` | ⚠️ | — | Yalnızca `Date Histogram` için |

---

## 📐 KURAL 9 — SOC Notes Bölümü

```markdown
**SOC notes**
- {Gözlem 1} → {aksiyon/anlam}.
- {Gözlem 2} → {aksiyon/anlam}.
- Correlate with: {ilişkili panel veya event referansı}.
- Severity: **{seviye}**
```

### Zorunlu İçerik

| Öğe | Zorunlu | Açıklama |
|-----|---------|----------|
| En az 2 analiz notu | ✅ | `→` ile nedeni ve aksiyonu bağla |
| Severity | ✅ | Aşağıdaki seviye tablosundan seç |
| Correlate with | ⚠️ | İlişkili panel/event varsa ekle |
| Operational threshold | ⚠️ | Spike/KPI panellerinde ekle |

### Severity Seviyeleri

| Seviye | Kullanım |
|--------|----------|
| `**Low**` | Bilgilendirme, baseline izleme |
| `**Medium**` | Normal dışı ancak doğrulanması gereken |
| `**Medium → High**` | Koşula bağlı yükseliyor |
| `**High**` | Aktif araştırma gerektirir |
| `**High → Critical**` | Bağlama göre kritik olabilir |
| `**Critical**` | Anında eskalasyon ve müdahale |

### SOC Notes Yazım Kuralları

1. Her not `- ` ile başlar (madde işareti).
2. `→` operatörü kullanarak **gözlem → anlam/aksiyon** bağlantısı kur.
3. Correlate referanslarında **Panel numarası** veya **Event ID** belirt.
4. Severity'de mutlaka `**bold**` format kullan.

### ✅ Doğru Örnekler
```markdown
**SOC notes**
- High frequency on a single host → investigate.
- New or unexpected accounts → possible escalation.
- Sudden privilege activity outside baseline → review 4624 and 4688 correlation.
- Severity: **High** if non-admin baseline user.
```

### ❌ Yanlış Örnekler
```
Severity:
High                     ← Format yok, analiz notu yok

Purpose:
Task-based persistence.
Severity:
High                     ← SOC notes bölümü yok, severity ayrı yazılmış
```

---

## 📐 KURAL 10 — Dashboard Footer

Her dashboard dosyası şu satırla bitmeli:

```markdown
---

END OF DASHBOARD-{N} TEMPLATE
```

---

## 📐 KURAL 11 — Özel Durumlar

### Data Table + Columns Listesi

Bazı panellerde bucket yerine doğrudan **kolon listesi** tercih edilebilir. Bu durumda:

```markdown
**Visualization:** Data Table

**Metrics**
- Aggregation: `Count`
- Custom Label: `{etiket}`

**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `{birincil_alan}`
   - Order by: `Count`
   - Order: `Descending`
   - Size: `{N}`
   - Custom Label: `{etiket}`

**Columns (display context)**
- `@timestamp`
- `agent.name`
- `{alan_1}`
- `{alan_2}`
```

> **Not:** `Columns` bölümü ek bağlam verir. OpenSearch'te Data Table visualization'da hangi kolonların görüntüleneceğini belirtir. Bu bölüm isteğe bağlıdır, ancak eklendiyse bucket'tan sonra gelmelidir.

### Stacked Bar Modu

Vertical Bar + Stacked kullanıldığında:

```markdown
**Visualization:** Vertical Bar  
Mode: **Stacked**
```

### Unique Count Metric

Normal Count yerine distinct değer sayısı gerektiğinde:

```markdown
**Metrics**
- Aggregation: `Unique Count`
- Field: `agent.name`
- Custom Label: `Distinct Target Hosts`
```

---

## 📐 KURAL 12 — Yasaklar

| Yasak | Neden |
|-------|-------|
| `KQL:` veya `KQL` etiketi kullanmak | Wazuh Indexer'da KQL çalışmaz, yalnızca DQL |
| `#` (h1) panel başlığı kullanmak | Paneller `##` (h2), dashboard başlığı `#` (h1) |
| SOC notes olmadan panel yazmak | Her panel SOC analistine rehberlik etmeli |
| Severity belirtmemek | Analist triage önceliklendirmesi için zorunlu |
| Bucket'sız chart/table yazmak | Metric (KPI) dışında her visualization bucket gerektirir |
| Sıralama kuralını bozmak | Bölüm sırası: Purpose → DQL → Visualization → Metrics → Buckets → SOC notes |
| Aynı DQL + aynı viz + aynı bucket kombinasyonunu başka panelde tekrarlamak | Overlap kurallarına bakınız |

---

## 🔄 Tam Panel Şablonu (Copy-Paste Ready)

Aşağıdaki şablonu kopyalayıp `{...}` alanlarını doldurun:

```markdown
## Panel {N} — SOC | {Kategori} | {İsim}

**Purpose:** {Tek cümle açıklama — neyi tespit ediyor, neden önemli.}

**DQL**
```dql
{sorgu}
```

**Visualization:** {Tip}

**Metrics**
- Aggregation: `{Count / Unique Count / Sum}`
- Field: `{alan}` ← Count dışında zorunlu
- Custom Label: `{anlamlı etiket}`

**Buckets**
- Aggregation: `Terms`
- Field: `{alan}`
- Order by: `Count`
- Order: `Descending`
- Size: `{N}`
- Custom Label: `{anlamlı etiket}`

**SOC notes**
- {Gözlem} → {anlam / aksiyon}.
- {Gözlem} → {anlam / aksiyon}.
- Correlate with: {Panel X / Event ID YYYY}.
- Severity: **{Low / Medium / High / Critical}**

---
```
