# 🛡️ Wazuh SOC Blueprint — Proje Analiz Raporu

**Analiz Tarihi:** 2 Mayıs 2026  
**Proje Versiyonu:** v1.0.0  
**Lisans:** MIT  
**Analiz Kapsamı:** Tüm dizin yapısı, 5 dashboard, 3 runbook, rules, ilm, mitre, slack-alerts

> [!IMPORTANT]
> Bu doküman projenin tamamına yapılmış kapsamlı bir analizdir. AI Agent'lar yeni iş yaparken bu bulguları dikkate almalı, çözülmüş sorunları tekrar raporlamamalı ve öncelik sıralamasına uymalıdır.

---

## 📁 Proje Yapısı

```
wazuh-soc-blueprint/
├── LICENSE                          (MIT)
├── README.md                        (3.5 KB — iyi dokümante)
├── dashboards/                      (5 dosya — ~20 KB toplam)
│   ├── SOC-Dashboard-1.md           (Auth & Correlation — 7 panel)
│   ├── SOC-Dashboard-2-RDP-Deep.md  (RDP Deep — 8 panel)
│   ├── SOC-Dashboard-3-Privilege-Escalation.md (PrivEsc — 4 panel)
│   ├── SOC-Dashboard-4-Persistence.md (Persistence — 5 panel)
│   └── SOC-Dashboard-5-Threat-Intel-IOC.md (TI/IOC — 5 panel)
├── docs/                            (AI Agent kuralları ve analiz)
│   ├── AI-Agent-Panel-Standard.md   (Panel yazım standardı)
│   ├── AI-Agent-Overlap-Rules.md    (Overlap tespit kuralları)
│   ├── Dashboard-4-5-Rewrite-Roadmap.md (D4-D5 yeniden yazım planı)
│   └── Project-Analysis.md          (Bu dosya)
├── rules/                           (Boş — sadece README.md)
│   └── README.md
├── runbooks/                        (3 dosya)
│   ├── Incident-Response-Runbook.md
│   ├── Brute-Force-Runbook.md
│   └── Privilege-Escalation-Runbook.md
├── slack-alerts/                    (1 dosya — placeholder)
│   └── Slack-Webhooks-Template.md
├── ilm/                             (1 dosya — placeholder)
│   └── Index-Lifecycle-Policy.md
└── mitre/                           (1 dosya — placeholder)
    └── MITRE-Coverage-Matrix.md
```

---

## 🎯 Genel Değerlendirme Matrisi

| Kategori | Puan | Durum |
|----------|------|-------|
| Dashboard Kalitesi (D1-D3) | ⭐⭐⭐⭐⭐ | Mükemmel — Production Ready |
| Dashboard Kalitesi (D4-D5) | ⭐⭐⭐ | Yetersiz — Standart dışı format |
| Dokümantasyon (README) | ⭐⭐⭐⭐ | İyi |
| Wazuh Kuralları | ⭐ | Kritik Eksik — Boş dizin |
| Runbook'lar | ⭐⭐⭐ | Orta — Template seviye |
| MITRE Kapsama | ⭐ | Placeholder |
| ILM Politikası | ⭐ | Placeholder |
| Slack Entegrasyonu | ⭐ | Placeholder |
| Otomasyon / CI/CD | ⭐ | Yok |
| **Genel Olgunluk** | **⭐⭐⭐** | **Orta** |

---

## ✅ Güçlü Yönler

### 1. Dashboard Mimarisi (D1-D3 — Çok Güçlü)
- **29 toplam panel** — 5 dashboard boyunca iyi dağıtılmış
- Her panel için **DQL sorguları** hazır (copy-paste ready)
- **Visualization tipi**, **metric tanımı**, **bucket konfigürasyonu** tam belirtilmiş
- **SOC notları** her panele eklenmiş — analist triage rehberliği sağlıyor
- **Severity seviyeleri** açıkça tanımlı

### 2. Detection Engineering Mantığı
- Machine account (`*$`) filtreleme tutarlı şekilde uygulanmış
- IPv6 / loopback gürültü filtrasyonu var
- Operasyonel eşikler belirlenmiş (5dk >50 fail, IP başına >15 fail)
- Fail→Success korelasyonu destekleniyor

### 3. RDP Deep Monitoring (Dashboard 2 — En Detaylı)
- 8 panel ile en kapsamlı dashboard
- **Internal subnet tracking** (10.38.1.*, 172.16.16.*)
- **SSL VPN IP pool** izleme (10.212.134.200-210 + fdff:ffff*)
- **Multi-host lateral movement** tespiti (unique count agent.name)
- **Public IP tripwire** — sertleştirilmiş ortamda 0 beklenen kural

### 4. README Kalitesi
- Mimari varsayımlar belirtilmiş (Wazuh 4.x, OpenSearch, Windows auditing)
- Event ID tablosu ve eşik örnekleri mevcut

---

## ⚠️ Tespit Edilen Sorunlar ve Eksiklikler

### 🔴 Kritik Seviye (P0)

#### BULGU-001: Wazuh Custom Rules Dizini Boş ✅ ÇÖZÜLDÜ — 04.05.2026
- **Konum:** `/rules/local_rules.xml`
- **Durum:** `local_rules.xml` dosyası oluşturuldu. `threat_intel`, `malware`, `dns` gibi dashboard bağımlılığı olan rule group tanımları ve temel threshold kuralları eklendi.
- **Öneri:** Gerçek bir ortama kurulum yapılırken bu kuralların `ossec.conf` içinde tanımlanması gerekir.

#### BULGU-002: Dashboard 4-5 Format Standardı Dışında ✅ ÇÖZÜLDÜ — 04.05.2026
- **Konum:** `/dashboards/SOC-Dashboard-4-Persistence.md` ve `SOC-Dashboard-5-Threat-Intel-IOC.md`
- **Durum:** `AI-Agent-Panel-Standard.md` kurallarına uygun şekilde tamamen yeniden yazıldı.
- **Detay:** `docs/Dashboard-4-5-Rewrite-Roadmap.md` içindeki tüm plan uygulandı.

#### BULGU-003: MITRE Coverage Matrix Placeholder ✅ ÇÖZÜLDÜ — 04.05.2026
- **Konum:** `/mitre/MITRE-Coverage-Matrix.md`
- **Durum:** Placeholder kaldırıldı, 5 dashboard ve custom rule'ları kapsayan detaylı matris oluşturuldu.
- **Detay:** Tactic bazlı özet, detaylı technique mapping ve eksik (gap) listesi eklendi.

---

### 🟡 Yüksek Seviye (P1)

#### BULGU-004: Slack Alert Entegrasyonu Placeholder ✅ ÇÖZÜLDÜ — 04.05.2026
- **Konum:** `/slack-alerts/Slack-Webhooks-Template.md`
- **Durum:** Placeholder kaldırıldı, `ossec.conf` konfigürasyonu ve JSON payload şablonu içeren tam rehber eklendi.

#### BULGU-005: ILM Politikası Placeholder ✅ ÇÖZÜLDÜ — 04.05.2026
- **Konum:** `/ilm/Index-Lifecycle-Policy.md`
- **Durum:** OpenSearch ISM (Index State Management) JSON politikası ve tier tanımları (Hot/Warm/Cold/Delete) eklendi.

#### BULGU-006: Saved Search / NDJSON Export Yok ✅ ÇÖZÜLDÜ — 04.05.2026
- **Durum:** `docs/OpenSearch-Import-Guide.md` oluşturuldu. Manuel kurulum ve NDJSON tabanlı otomasyon (IaC) stratejisi tanımlandı.

---

### 🟡 Orta Seviye (P2)

#### BULGU-007: Runbook'lar Template Seviyesinde
- **Konum:** `/runbooks/`
- **Eksikler:**
  - Kontakt bilgileri / eskalasyon matrisi yok
  - Araç spesifik komutlar yok (ör. `wazuh-control`, `agent_groups`)
  - Zaman SLA'ları tanımlı değil
  - Containment playbook'ları detaylandırılmamış
- **Öneri:** Wazuh CLI komutları, eskalasyon matrisi ve SLA tanımları eklenmeli

#### BULGU-008: Linux / macOS Endpoint Kapsamı Yok
- **Durum:** Tüm dashboard'lar `data.win.*` field'larına odaklanmış
- **Etki:** Linux syslog, auditd, macOS endpoint coverage sıfır
- **Öneri:** Gereksinim varsa ayrı Linux SOC Dashboard oluşturulmalı

#### BULGU-009: Execution Detection Gap
- **Durum:** T1059 (PowerShell, cmd), T1055 (Process Injection), T1003 (Credential Dumping) için dashboard paneli yok
- **Öneri:** Dashboard-6 (Execution & Process Monitoring) oluşturulması düşünülmeli

---

### 🟢 Düşük Seviye (P3)

#### BULGU-010: `.gitignore` Dosyası Eksik
#### BULGU-011: `CONTRIBUTING.md` / `CHANGELOG.md` Yok
#### BULGU-012: CI/CD Pipeline Yok (markdown lint, link validation vb.)

---

## 🗺️ MITRE ATT&CK Kapsama Analizi

### Mevcut Kapsama

| MITRE Technique | Tactic | Dashboard | Panel | Durum |
|----------------|--------|-----------|-------|-------|
| T1110 — Brute Force | Credential Access | D1 | P1-P5 | ✅ Güçlü |
| T1110.001 — Password Guessing | Credential Access | D1 | P3 | ✅ |
| T1110.003 — Password Spraying | Credential Access | D1 | P4 | ✅ |
| T1078 — Valid Accounts | Defense Evasion | D1 | P6-P7 | ✅ |
| T1021.001 — Remote Desktop Protocol | Lateral Movement | D2 | P1-P8 | ✅ Mükemmel |
| T1021.002 — SMB/Windows Admin Shares | Lateral Movement | D4 | P3 | ⚠️ Kısmi |
| T1078.002 — Domain Accounts | Persistence | D3 | P1-P3 | ✅ |
| T1098 — Account Manipulation | Persistence | D3 | P2-P3 | ✅ |
| T1053.005 — Scheduled Task | Persistence | D4 | P1 | ✅ |
| T1547.001 — Registry Run Keys | Persistence | D4 | P2 | ✅ |
| T1543.003 — Windows Service | Persistence | D4 | P5 | ✅ |
| T1550.002 — Pass the Hash | Credential Access | D4 | P4 | ⚠️ Kısmi |
| T1071 — Application Layer Protocol | C2 | D5 | P2 | ⚠️ Kısmi |
| T1568 — Dynamic Resolution (DNS) | C2 | D5 | P4 | ⚠️ Kısmi |

### Tespit Edilen Kapsama Boşlukları

| MITRE Technique | Tactic | Önem | Önerilen Aksiyon |
|----------------|--------|------|-----------------|
| T1059 — Command and Scripting Interpreter | Execution | 🔴 Yüksek | Dashboard-6 paneli oluştur (4688 + Sysmon EID 1) |
| T1059.001 — PowerShell | Execution | 🔴 Yüksek | Dashboard-6 paneli oluştur (4104 ScriptBlock Logging) |
| T1055 — Process Injection | Defense Evasion | 🔴 Yüksek | Sysmon EID 8/10 paneli gerekli |
| T1003 — OS Credential Dumping | Credential Access | 🔴 Yüksek | LSASS erişim izleme (Sysmon EID 10) |
| T1486 — Data Encrypted for Impact | Impact | 🔴 Yüksek | FIM + yüksek hacimli dosya değişikliği paneli |
| T1569.002 — Service Execution | Execution | 🟡 Orta | D4-P5 ile kısmen kapsanıyor, 4697 eklenebilir |
| T1036 — Masquerading | Defense Evasion | 🟡 Orta | Process name anomaly paneli |
| T1070 — Indicator Removal | Defense Evasion | 🟡 Orta | Event log clearing (1102) paneli |
| T1105 — Ingress Tool Transfer | C2 | 🟡 Orta | Outbound connection anomaly |
| T1219 — Remote Access Software | C2 | 🟡 Orta | Known RAT process detection |

---

## 📊 Dashboard Kalite Karşılaştırması

| Dashboard | Panel Sayısı | Format Kalitesi | DQL Detayı | SOC Notları | Genel |
|-----------|-------------|-----------------|------------|-------------|-------|
| D1 — Auth & Correlation | 7 | ⭐⭐⭐⭐⭐ | Tam | Detaylı | 🟢 Production Ready |
| D2 — RDP Deep | 8 | ⭐⭐⭐⭐⭐ | Tam | Detaylı | 🟢 Production Ready |
| D3 — PrivEsc | 4 | ⭐⭐⭐⭐⭐ | Tam | Detaylı | 🟢 Production Ready |
| D4 — Persistence | 5 | ⭐⭐⭐ | Kısmi | Minimum | 🟡 Yeniden Yazım Gerekli |
| D5 — Threat Intel | 5 | ⭐⭐⭐ | Kısmi | Minimum | 🟡 Yeniden Yazım Gerekli |

---

## 🚀 İyileştirme Önceliklendirme Tablosu

| Öncelik | Bulgu ID | İş Kalemi | İlgili Docs |
|---------|----------|-----------|-------------|
| **P0** | BULGU-002 | Dashboard 4-5 Yeniden Yazım | `Dashboard-4-5-Rewrite-Roadmap.md` |
| **P0** | BULGU-001 | Wazuh Custom Rules Oluşturma | — |
| **P0** | BULGU-003 | MITRE Coverage Matrix Doldurma | Bu dosyadaki MITRE tablosu |
| **P1** | BULGU-004 | Slack Integration Gerçek Konfigürasyonu | — |
| **P1** | BULGU-005 | ILM/ISM Politikası JSON | — |
| **P1** | BULGU-006 | NDJSON Export Oluşturma | — |
| **P2** | BULGU-009 | Execution Detection Dashboard (D6) ✅ ÇÖZÜLDÜ — 04.05.2026 | — |
| **P2** | BULGU-007 | Runbook Derinleştirme ✅ ÇÖZÜLDÜ — 04.05.2026 | — |
| **P2** | BULGU-008 | Linux/macOS Kapsamı | — |
| **P3** | BULGU-010/11 | Repo Hijyeni (.gitignore, CONTRIBUTING) ✅ ÇÖZÜLDÜ — 04.05.2026 | — |

---

## 📎 İlgili Dokümanlar

| Doküman | Amaç |
|---------|------|
| `docs/AI-Agent-Panel-Standard.md` | Panel yazım standart kuralları (12 kural) |
| `docs/AI-Agent-Overlap-Rules.md` | Panel overlap tespit kuralları ve 29-panel kayıt defteri |
| `docs/Dashboard-4-5-Rewrite-Roadmap.md` | D4-D5 yeniden yazım yol haritası ve kalite kontrol listesi |

---

## 📝 AI Agent İçin Talimatlar

1. **Yeni iş yaparken** bu dosyadaki bulgu listesini kontrol edin. Çözülmüş bir sorunu tekrar raporlamayın.
2. **Dashboard paneli yazarken** `AI-Agent-Panel-Standard.md` kurallarına uyun.
3. **Yeni panel eklerken** `AI-Agent-Overlap-Rules.md` içindeki kayıt defterini kontrol edin, overlap varsa uyarı verin.
4. **D4-D5 yeniden yazımında** `Dashboard-4-5-Rewrite-Roadmap.md` planını takip edin.
5. **MITRE gap'leri kapatırken** bu dosyadaki kapsama boşlukları tablosunu referans alın.
6. **Önceliklendirme** için bu dosyadaki P0→P3 sıralamasına uyun.
7. **Bir bulgu çözüldüğünde** bu dosyada ilgili bulgunun yanına `✅ ÇÖZÜLDÜ — {tarih}` notu ekleyin.
