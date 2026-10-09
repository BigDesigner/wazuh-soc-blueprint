# Project Task Pipeline & Roadmap

**Active Sprint:** `v1.0.0-soc-standardization`  
**Current State:** Sentinel Agent Memory Bank Bootstrapped  
**Status:** In Progress  

---

## 🚀 Immediate Next Actions (Sprint Backlog)

- [x] **TASK-000**: Remediate all defects, DQL errors, and missing SOC notes across Dashboards 1–6 per audit `7583260`.
- [x] **TASK-001**: Synchronize `README.md` to include Dashboard-6 (Execution & Process Monitoring) in file structure, dashboard overview, and Event ID tables.
- [x] **TASK-002**: Address high-priority MITRE gaps in `rules/local_rules.xml`:
  - `T1486 / T1490`: Added FIM/vssadmin shadow copy deletion rules (100600 & 100601).
  - `T1070.001`: Added Windows Event log clearing alerts for Event IDs 1102 & 104 (100500 & 100501).
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
