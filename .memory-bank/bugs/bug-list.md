# Active Bug & Coverage Gap Registry

| ID | Category | Severity | Confidence | Status | Description | Suggested Next Action |
|---|---|---|---|---|---|---|
| **BUG-001** | Architecture Gap | Medium | Verified | Open | Framework focuses exclusively on Windows/Sysmon telemetry (`data.win.*`). Linux auditd, SSH brute-force, and sudo abuse are unmonitored. | Design and create `SOC-Dashboard-7-Linux-Endpoint.md` and Linux Wazuh rules. |
| **BUG-002** | DevOps / CI/CD | Low | Verified | Open | No GitHub Actions workflow exists to automate markdown linting, DQL query validation, or XML syntax checking. | Implement `.github/workflows/validate.yml` checking XML syntax and DQL rules. |
| **BUG-003** | Documentation | Low | Verified | Closed | `README.md` file structure and dashboard coverage table omitted `SOC-Dashboard-6-Execution-Process.md`. | Resolved: Synchronized README.md with Dashboard 6 and Sysmon EIDs. |
| **BUG-004** | Automation / IaC | Low | Inferred | Open | `dashboards/exports/` does not yet contain pre-generated OpenSearch `.ndjson` files for one-click import. | Generate or export `.ndjson` bundles for Dashboards 1–6. |
| **GAP-001** | MITRE Detection | High | Verified | Open | **T1486 (Data Encrypted for Impact)**: Ransomware mass-file encryption or shadow copy deletion not monitored via FIM. | Add FIM rule and vssadmin alert in `local_rules.xml`. |
| **GAP-002** | MITRE Detection | Medium | Verified | Open | **T1070 (Indicator Removal)**: Event log clearing (Security 1102, System 104) lacks dedicated alert rule. | Add Event ID 1102 / 104 monitoring rule to `local_rules.xml`. |
| **GAP-003** | MITRE Detection | Medium | Verified | Open | **T1219 (Remote Access Software)**: Known unauthorized remote desktop and RAT processes (AnyDesk, TeamViewer) unmonitored. | Add process detection for unauthorized remote access tools. |
