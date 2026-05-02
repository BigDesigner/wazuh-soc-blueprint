# 🔍 AI Agent — Panel Overlap Tespit Kuralları

**Versiyon:** 1.0  
**Amaç:** AI Agent, yeni bir panel yazarken veya mevcut panelleri gözden geçirirken bu kuralları uygulayarak **çakışan, gereksiz veya tekrarlayan** panelleri tespit edecek ve kullanıcıyı uyaracaktır.

---

## 📌 Overlap Nedir?

İki panel aşağıdaki koşullardan birini karşıladığında "overlap" sayılır:

| Tip | Tanım | Eylem |
|-----|-------|-------|
| **Tam Çakışma** | Aynı DQL + aynı Visualization + aynı Bucket | ❌ Birini sil |
| **Kısmi Çakışma** | Aynı DQL ama farklı Visualization veya Bucket | ⚠️ Uyar, gerekçe iste |
| **Mantıksal Çakışma** | Farklı DQL ama aynı security soruyu cevaplıyor | ⚠️ Uyar, birleştirme öner |
| **Tamamlayıcı** | Aynı veri kaynağı ama farklı analiz açısı | ✅ Kabul, ilişkiyi belirt |

---

## 🔴 Overlap Tespit Algoritması

AI Agent, her yeni panel talebi geldiğinde şu adımları izlemelidir:

### Adım 1 — DQL Karşılaştırma

```
Yeni panelin DQL sorgusundaki temel filtreleri çıkar:
  → EventID (ör. 4624, 4625, 4698, 7045)
  → rule.groups (ör. authentication_failed, threat_intel)
  → logonType (ör. 3, 10)
  → authenticationPackageName (ör. NTLM)

Bu filtreleri mevcut tüm panellerle karşılaştır.
```

### Adım 2 — Visualization + Bucket Karşılaştırma

```
Aynı DQL temeli paylaşan paneller bulunduysa:
  → Visualization tipi aynı mı?
  → Bucket field'ları aynı mı?
  → Her ikisi de evet → TAM ÇAKIŞMA
  → Biri farklı → KISMİ ÇAKIŞMA
```

### Adım 3 — Karar ve Uyarı

```
TAM ÇAKIŞMA:
  → "⚠️ OVERLAP: Panel X (Dashboard Y) ile bu panel aynı veriyi,
     aynı görselleştirmeyle gösteriyor. Biri gereksiz."

KISMİ ÇAKIŞMA:
  → "ℹ️ PARTIAL OVERLAP: Panel X (Dashboard Y) aynı veri kaynağını
     kullanıyor ama {farklılık}. Her ikisi de gerekli mi?"

MANTIKSAL ÇAKIŞMA:
  → "ℹ️ LOGICAL OVERLAP: Panel X aynı güvenlik sorusunu farklı
     açıdan cevaplıyor. Birleştirme düşünülebilir."

TAMAMLAYICI:
  → "✅ COMPLEMENTARY: Panel X ilişkili veriyi farklı açıdan analiz
     ediyor. Cross-reference notu ekleniyor."
```

---

## 📊 Mevcut Dashboard Overlap Analizi

### Dashboard 1-5 Arası Tespit Edilen Overlap'ler

---

### OVERLAP #1 — D1-P7 vs D4-P3 ⚠️ Kısmi Çakışma

| Özellik | D1-P7 | D4-P3 |
|---------|-------|-------|
| **Panel** | SOC \| Network Logons \| Success (Users Only) | SMB Logon Spike (Type 3) |
| **DQL** | `rule.groups:authentication_success AND data.win.eventdata.logonType:3` | `rule.groups:authentication_success AND data.win.eventdata.logonType:3` |
| **Viz** | Horizontal Bar | Line |
| **Bucket** | Terms → targetUserName | Date Histogram → @timestamp (5m) |
| **Durum** | **Aynı DQL, farklı visualization** | — |

**Analiz:**  
Bu iki panel **tamamlayıcıdır** (complementary). D1-P7 "hangi kullanıcı" sorusunu cevaplarken, D4-P3 "ne zaman spike oldu" sorusunu cevaplar. **Her ikisi de gerekli**, ancak D4-P3 yeniden yazılırken cross-reference notu eklenmeli:

```
SOC notes:
- Correlate with: Dashboard-1 Panel 7 (user-level SMB baseline).
```

**Karar:** ✅ Her ikisi kalacak, D4-P3'e cross-reference eklenecek.

---

### OVERLAP #2 — D1-P2 vs D2-P2 ⚠️ Kısmi Çakışma

| Özellik | D1-P2 | D2-P2 |
|---------|-------|-------|
| **Panel** | SOC \| Top \| Source IP (Failed Logins) | SOC \| RDP \| Failed \| Top Source IP |
| **DQL** | `rule.groups:authentication_failed AND ipAddress:*` | `data.win.system.eventID:4625 AND ipAddress:*` |
| **Viz** | Horizontal Bar | Horizontal Bar |
| **Bucket** | Terms → ipAddress (Size 15) | Terms → ipAddress (Size 15) |
| **Durum** | **Çok benzer DQL, aynı viz + bucket** | — |

**Analiz:**  
D1-P2 tüm `authentication_failed` rule group'unu yakalar (Event 4625 + diğer fail olayları). D2-P2 yalnızca EventID 4625'e odaklanır. Büyük ölçüde aynı veriyi gösterirler.

**Neden kabul edilebilir:**  
D2, **RDP-odaklı** bir dashboard. RDP analistinin Dashboard-1'e geçmeden aynı veriyi kendi bağlamında görmesi operasyonel avantaj sağlar. Ancak bu bir **bilinçli tasarım kararıdır** — yeni paneller eklerken bu tarz duplikasyondan kaçınılmalıdır.

**Karar:** ✅ Kabul (bilinçli — dashboard kontekst bağımsızlığı için). Yeni panellerde tekrarlamayın.

---

### OVERLAP #3 — D3-P1 Severity Context vs D1-P6 Auth Correlation ⚠️ Mantıksal Çakışma (Zayıf)

| Özellik | D3-P1 | D1-P6 |
|---------|-------|-------|
| **Panel** | SOC \| Special Privileges Assigned | SOC \| Correlation \| Failed vs Success |
| **DQL** | `eventID:4672` | `rule.groups:(auth_failed OR auth_success)` |

**Analiz:**  
Farklı veri kaynakları, farklı sorular. D3-P1'in SOC notes'unda "review 4624 and 4688 correlation" notu var — bu da D1-P6'ya atıfta bulunuyor. **Tamamlayıcı ilişki doğru şekilde belgelenmiş.**

**Karar:** ✅ Overlap değil — doğru cross-reference mevcut.

---

### OVERLAP #4 — D4-P5 vs D3 Genel ⚠️ Mantıksal Çakışma (Zayıf)

| Özellik | D4-P5 | D3 (Genel) |
|---------|-------|------------|
| **Panel** | Service Creation (7045) | Privilege Escalation Dashboard |
| **EventID** | 7045 | 4672, 4728, 4732, 4756 |

**Analiz:**  
Service creation (7045) hem **persistence** hem **privilege escalation** bağlamında kullanılabilir. D4'te persistence olarak konumlandırılmış, D3'te ise 4697 (Service Installed) yok. Bu mantıklı bir ayrımdır.

**Karar:** ✅ Kabul. D4-P5 persistence bağlamında kalacak. Gerekirse D3'e 4697 (Service Installed — Event-level) ayrı panel eklenebilir.

---

## 🛑 AI Agent Uyarı Şablonları

Agent, overlap tespit ettiğinde aşağıdaki şablonları kullanmalıdır:

### Tam Çakışma Uyarısı

```
⚠️ OVERLAP TESPİT EDİLDİ

Yazmak istediğiniz panel, mevcut bir panel ile tam çakışıyor:

  Mevcut : Dashboard-{X}, Panel {Y} — "{Panel Adı}"
  DQL    : {aynı sorgu}
  Viz    : {aynı tip}
  Bucket : {aynı field}

Bu iki panel aynı veriyi aynı şekilde gösterecek.
Önerim: Mevcut paneli kullanın veya farklı bir analiz açısı belirleyin.

İki paneli de tutmak istiyor musunuz? (Evet/Hayır)
```

### Kısmi Çakışma Uyarısı

```
ℹ️ KISMİ OVERLAP

Yazmak istediğiniz panel, mevcut bir panel ile aynı veri kaynağını paylaşıyor:

  Mevcut     : Dashboard-{X}, Panel {Y} — "{Panel Adı}"
  Ortak DQL  : {paylaşılan koşullar}
  Farklılık  : {ne farklı — viz tipi, bucket, ek filtre}

Her ikisi de farklı bir soruyu cevaplıyorsa tamamlayıcıdır.
Aynı soruyu cevaplıyorsa biri gereksizdir.

Devam edeyim mi? Cross-reference notu da ekleyeceğim.
```

### Mantıksal Çakışma Uyarısı

```
ℹ️ MANTIKSAL OVERLAP

Bu panel ile mevcut bir panel benzer güvenlik sorusunu farklı yollarla cevaplıyor:

  Mevcut   : Dashboard-{X}, Panel {Y} — "{Panel Adı}"
  Soru     : "{her ikisinin cevapladığı soru}"
  Yaklaşım : Panel A → {yaklaşım}, Panel B → {yaklaşım}

Birleştirme mümkün mü? Yoksa her ikisi de farklı senaryolarda mı kullanılıyor?
```

---

## 📋 AI Agent — Yeni Panel Yazarken Kontrol Listesi

Her yeni panel talebi geldiğinde AI Agent şu listeyi sırasıyla kontrol etmelidir:

```
□ 1. DQL sorgusu Dashboard 1-5'teki herhangi bir panelle aynı EventID/rule.groups
     kullanıyor mu?
     → Evet: Overlap kontrolüne geç
     → Hayır: Devam et

□ 2. Aynı DQL temeli bulunduysa → Visualization + Bucket aynı mı?
     → Tam aynı: TAM ÇAKIŞMA uyarısı ver
     → Farklı: Farklılığı belirt, tamamlayıcı mı kontrol et

□ 3. Panel, mevcut bir panelle aynı güvenlik sorusunu mu cevaplıyor?
     → Evet: MANTIKSAL ÇAKIŞMA uyarısı ver
     → Hayır: Devam et

□ 4. Tüm kontroller geçildiyse → Paneli yaz

□ 5. Cross-reference gerekiyorsa → SOC notes'a ekle
```

---

## 📊 Mevcut Panel Kayıt Defteri (Quick Reference)

AI Agent, overlap kontrolü için bu tabloyu referans almalıdır:

| Dashboard | Panel | EventID / rule.groups | LogonType | Viz | Birincil Bucket Field |
|-----------|-------|----------------------|-----------|-----|----------------------|
| D1-P1 | KPI Failed Logins | authentication_failed | — | Metric | — |
| D1-P2 | Top Source IP (Failed) | authentication_failed | — | H.Bar | ipAddress |
| D1-P3 | Top Target Users (Failed) | authentication_failed | — | V.Bar | targetUserName |
| D1-P4 | Spike Failed (5m) | authentication_failed | — | Line | @timestamp (5m) |
| D1-P5 | Source IP Spike (5m) | authentication_failed | — | Bar | ipAddress + @timestamp |
| D1-P6 | Failed vs Success | auth_failed + auth_success | — | Table | targetUserName + rule.groups |
| D1-P7 | Network Logons Success | authentication_success | 3 | H.Bar | targetUserName |
| D2-P1 | RDP Success Users | 4624 | 10 | H.Bar | targetUserName |
| D2-P2 | RDP Failed Source IP | 4625 | — | H.Bar | ipAddress |
| D2-P3 | Auth Logon Type Dist. | 4624+LT10 OR 4625 | 10 | Table | ipAddress → targetUserName → eventID |
| D2-P4 | RDP Internal Timeline | 4624 | 10 | V.Bar Stacked | @timestamp (3h) + eventID |
| D2-P5 | RDP Internal IP→User→Host | 4624 | 10 | Table | ipAddress → targetUserName → agent.name |
| D2-P6 | RDP VPN IP→User→Host | 4624 | 10 | Table | ipAddress → targetUserName → agent.name |
| D2-P7 | VPN Multi-Host Count | 4624 | — | Table | ipAddress (Unique Count agent.name) |
| D2-P8 | RDP Public IP Tripwire | 4624 | 10 | H.Bar | ipAddress (NOT internal) |
| D3-P1 | Special Privileges | 4672 | — | Table | agent.name → subjectUserName |
| D3-P2 | Admin Group Changes | 4728/4732/4756 | — | Table | agent.name → TargetUserName |
| D3-P3 | Account Creation | 4720 | — | Table | agent.name → targetUserName |
| D3-P4 | Source IP Privilege | 4672/4728/4732/4756 | — | H.Bar | ipAddress |
| D4-P1 | Scheduled Tasks | 4698 | — | Table | *(mevcut: bucket yok)* |
| D4-P2 | Registry Run Keys | registry + Run/RunOnce | — | Table | *(mevcut: bucket yok)* |
| D4-P3 | SMB Logon Spike | authentication_success | 3 | Line | @timestamp (5m) |
| D4-P4 | NTLM Logons | 4624 + NTLM | — | Table | *(mevcut: bucket yok)* |
| D4-P5 | Service Creation | 7045 | — | Table | *(mevcut: bucket yok)* |
| D5-P1 | Threat Intel Matches | threat_intel | — | Table | *(mevcut: bucket yok)* |
| D5-P2 | Suspicious Outbound | firewall | — | H.Bar | destip |
| D5-P3 | Malware Hash | malware | — | Table | *(mevcut: bucket yok)* |
| D5-P4 | DNS Suspicious TLD | dns | — | Table | *(mevcut: bucket yok)* |
| D5-P5 | MITRE Distribution | mitre.id | — | Pie | rule.mitre.id |
