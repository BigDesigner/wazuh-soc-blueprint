# Project Task Pipeline & Roadmap

**Active Sprint:** `v1.0.0-soc-standardization`  
**Current State:** Sentinel Agent Memory Bank Bootstrapped  
**Status:** In Progress  

---

## 🚀 Immediate Next Actions (Sprint Backlog)

- [ ] **TASK-001**: Synchronize `README.md` to include Dashboard-6 (Execution & Process Monitoring) in file structure, dashboard overview, and Event ID tables.
- [ ] **TASK-002**: Address high-priority MITRE gaps in `rules/local_rules.xml`:
  - `T1486`: Add FIM/vssadmin shadow copy deletion alert.
  - `T1070`: Add Windows Event log clearing alert (Event ID 1102 / 104).
- [ ] **TASK-003**: Design `SOC-Dashboard-7-Linux-Endpoint.md` covering SSH brute-force, `sudo` privilege escalation, and auditd system telemetry.
- [ ] **TASK-004**: Generate OpenSearch NDJSON export bundles in `dashboards/exports/` for automated one-click import.
- [ ] **TASK-005**: Create GitHub Actions CI workflow (`.github/workflows/validate.yml`) validating XML rules schema and markdown links.

---

## 📋 Comprehensive Backlog

### Phase 1: Endpoint Expansion (Linux / Container)
- Linux Auditd rules for command execution and file modification.
- Docker / Kubernetes container security monitoring blueprint.

### Phase 2: Threat Detection & Evasion Hardening
- LOLBAS process creation monitoring refinement.
- Memory injection & DLL unhooking alerting (Sysmon Event ID 7 & 8).
- Remote Access Tool (RAT) process monitoring (`T1219`).

### Phase 3: Automation & CI/CD
- Python-based DQL query syntax validator script.
- Automatic NDJSON bundle packager.

---

## 🚧 Blockers & Impediments
- None currently identified.
