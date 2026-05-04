# 🚑 Incident Response Master Runbook — Wazuh SOC

**Version:** 1.0  
**Status:** Production Ready  
**Scope:** Standardized response flow for alerts triggered via Wazuh SOC Blueprints.

---

## 🚦 1. Triage & Identification (Phase 1)

When an alert is received, follow this strict investigative order using the Wazuh Dashboard and CLI.

### 1.1 Scope Definition
- **Hosts:** Identify all affected `agent.name`.
- **Users:** List all `data.win.eventdata.targetUserName`.
- **Network:** Extract `data.win.eventdata.ipAddress` (Source) and `data.destip` (Destination).

### 1.2 Timeline Analysis
- Filter by `agent.id` and the specific `alert.id`.
- Check for "Spike vs. Baseline" using Dashboard-1 (Auth) or Dashboard-4 (SMB Spike).

### 1.3 Wazuh CLI Helper Commands
```bash
# Check agent status and OS version
/var/ossec/bin/agent_control -i <AGENT_ID>

# List active responses currently running on the agent
/var/ossec/bin/agent_control -L

# Verify rule details that triggered the alert
grep "id=\"<RULE_ID>\"" /var/ossec/ruleset/rules/*.xml
```

---

## ⚖️ 2. Escalation Matrix

| Incident Type | Initial Severity | Escalation Path | SLA (Response) |
|---------------|------------------|-----------------|----------------|
| **Brute Force (Failures)** | Medium | Tier 1 Analyst | 30 mins |
| **Auth Success from Unknown IP**| High | Tier 2 / SOC Lead | 15 mins |
| **Privileged Group Change** | Critical | SOC Manager / CISO | 5 mins |
| **Malware / C2 Beaconing** | Critical | IR Team / NetSec | 10 mins |

---

## 🛡️ 3. Containment & Eradication (Phase 2)

### 3.1 Host Isolation (Wazuh Active Response)
If the compromise is confirmed, use Wazuh Active Response to isolate the host.
```bash
# Trigger a custom script to block a source IP at the agent firewall
/var/ossec/bin/agent_control -b <SOURCE_IP> -f <SCRIPT_NAME> -u <AGENT_ID>
```

### 3.2 Account Lockdown
- Reset passwords for all affected `targetUserName`.
- Disable compromised accounts until full forensic analysis is complete.
- Revoke all active sessions (Kerberos tickets/NTLM).

---

## 🔍 4. Forensic Evidence Checklist

- [ ] **Process Tree:** Review Dashboard-6 for parent/child process anomalies.
- [ ] **Network:** Check Dashboard-5 for matches against known C2/Malware IOCs.
- [ ] **Persistence:** Search for newly created Services (7045) or Scheduled Tasks (4698).
- [ ] **Logs:** Export relevant alerts in `.ndjson` or `.csv` for external investigation.

---

## 📝 5. Post-Incident Activities (Phase 3)

1. **Root Cause Analysis (RCA):** How did the initial access occur? (Phishing, Exploit, Misconfig).
2. **Detection Tuning:** Do we need to adjust thresholds in `local_rules.xml`?
3. **Reporting:** Fill the SOC Incident Report Template including MITRE technique IDs involved.

---

END OF INCIDENT RESPONSE RUNBOOK
