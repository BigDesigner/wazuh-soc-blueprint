# MITRE ATT&CK Coverage Matrix — Wazuh SOC Blueprint

**Version:** 1.1  
**Last Updated:** 09.10.2026  
**Scope:** Dashboards 1-6 & `rules/local_rules.xml`  

---

## 📊 Coverage Summary

| Tactic | Techniques Covered | Coverage Level |
|--------|-------------------|----------------|
| **Credential Access** | T1110, T1110.001, T1110.003, T1003, T1550.002 | 🟢 High |
| **Lateral Movement** | T1021.001, T1021.002 | 🟢 High |
| **Persistence** | T1078, T1098, T1053.005, T1547.001, T1543.003 | 🟢 High |
| **Defense Evasion** | T1078, T1036, T1055, T1070.001 | 🟢 High |
| **Command and Control** | T1071, T1568 | 🟡 Medium |
| **Execution** | T1059, T1059.001, T1569.002 | 🟢 High (D6 added) |
| **Impact** | T1486, T1490 | 🟢 High (Rules 100600 & 100601 added) |

---

## 🛠️ Detailed Technique Mapping

| Technique ID | Technique Name | Tactic | Source | Dashboard / Rule Reference | Status |
|--------------|----------------|--------|--------|----------------------------|--------|
| **T1110** | Brute Force | Credential Access | Wazuh Rule 100300 | D1 — P1, P2, P3, P4, P5 | ✅ Full |
| **T1110.001**| Password Guessing | Credential Access | EventID 4625 | D1 — P3 | ✅ Full |
| **T1110.003**| Password Spraying | Credential Access | EventID 4625 | D1 — P4 | ✅ Full |
| **T1059** | Command and Scripting | Execution | EventID 4104 / 4688 | D6 — P1, P2, P5 | ✅ Full |
| **T1059.001**| PowerShell | Execution | EventID 4104 | D6 — P1 | ✅ Full |
| **T1003** | OS Credential Dumping | Credential Access | Sysmon EID 10 | D6 — P3 | ✅ Full |
| **T1055** | Process Injection | Defense Evasion | Sysmon EID 8/25 | D6 — P4 | ✅ Full |
| **T1070.001**| Clear Windows Event Logs | Defense Evasion | EventID 1102 / 104 | Wazuh Rules 100500, 100501 | ✅ Full |
| **T1078** | Valid Accounts | Defense Evasion / Persistence | EventID 4624 | D1 — P6, P7 | ✅ Full |
| **T1021.001**| Remote Desktop Protocol | Lateral Movement | EventID 4624 (LT10) | D2 — All Panels (P1-P8) | ✅ Full |
| **T1021.002**| SMB/Windows Admin Shares| Lateral Movement | EventID 4624 (LT3) | D4 — P3 | ⚠️ Partial |
| **T1078.002**| Domain Accounts | Persistence | EventID 4720/4732 | D3 — P1, P2, P3 | ✅ Full |
| **T1098** | Account Manipulation | Persistence | EventID 4738 | D3 — P2, P3 | ✅ Full |
| **T1053.005**| Scheduled Task | Persistence | EventID 4698 | D4 — P1 | ✅ Full |
| **T1547.001**| Registry Run Keys | Persistence | FIM / Rule 100200 | D4 — P2 | ✅ Full |
| **T1543.003**| Windows Service | Persistence | EventID 7045 | D4 — P5 | ✅ Full |
| **T1550.002**| Pass the Hash | Credential Access | EventID 4624 (NTLM) | D4 — P4 | ⚠️ Partial |
| **T1071** | Application Layer Protocol | Command and Control | Firewall Logs | D5 — P2 | ⚠️ Partial |
| **T1568** | Dynamic Resolution | Command and Control | DNS Logs / Rule 100100 | D5 — P4 | ⚠️ Partial |
| **T1036** | Masquerading | Defense Evasion | Sysmon / Rule 100001 | D5 — P3 | 🟡 Planned |
| **T1486** | Data Encrypted for Impact | Impact | FIM / Rule 100601 | Wazuh Rule 100601 (Mass FIM) | ✅ Full |
| **T1490** | Inhibit System Recovery | Impact / Defense Evasion | EventID 4688 / Sysmon 1 | Wazuh Rule 100600 (vssadmin/bcdedit) | ✅ Full |

---

## 🚫 Identified Gaps (Roadmap)

| Technique ID | Tactic | Priority | Recommended Action |
|--------------|--------|----------|-------------------|
| **T1219** | Remote Access Software | 🟡 Medium | Known RAT process detection (AnyDesk, TeamViewer) |

---

> [!TIP]
> This matrix is synchronized with `dashboards/` and `rules/local_rules.xml`.
