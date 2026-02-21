![Wazuh](https://img.shields.io/badge/Wazuh-4.x-blue)
![SIEM](https://img.shields.io/badge/SIEM-Detection%20Engineering-red)
![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK-purple)
![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen)
![Version](https://img.shields.io/badge/Version-v1.0.0-black)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

<p align="center">
  <h1 align="center">Wazuh SOC Blueprint</h1>
  <p align="center">
    Enterprise-Grade Detection Engineering Framework
    <br />
    Authentication • RDP • Privilege Escalation • Persistence • Threat Intel
  </p>
</p>

---

# Wazuh SOC Blueprint
Enterprise-Grade Detection Engineering & SOC Dashboard Framework

---

## Overview

This repository contains a production-ready Security Operations Center (SOC) blueprint built on top of Wazuh.

It includes:

- Structured SOC dashboards (copy/paste-ready)
- Detection engineering logic and SOC thresholds
- Privilege escalation monitoring
- RDP & authentication abuse detection
- Persistence & lateral movement tracking
- Threat intelligence/IOC monitoring templates
- Escalation guidance and SOC triage notes
- MITRE ATT&CK mapping overview

---

## Architecture Assumptions

- Wazuh Manager 4.x
- Wazuh Indexer (OpenSearch)
- Index pattern: `wazuh-alerts-*`
- Windows auditing enabled (Advanced Audit Policy)
- Sysmon recommended
- Cluster health: Green

---

## Repository Structure

```
/dashboards
   SOC-Dashboard-1.md
   SOC-Dashboard-2-RDP-Deep.md
   SOC-Dashboard-3-Privilege-Escalation.md
   SOC-Dashboard-4-Persistence.md
   SOC-Dashboard-5-Threat-Intel-IOC.md

/rules
   README.md

/runbooks
   Incident-Response-Runbook.md
   Brute-Force-Runbook.md
   Privilege-Escalation-Runbook.md

/slack-alerts
   Slack-Webhooks-Template.md

/ilm
   Index-Lifecycle-Policy.md

/mitre
   MITRE-Coverage-Matrix.md
```

---

## Dashboard Coverage

| Dashboard | Focus Area |
|-----------|------------|
| Dashboard-1 | Authentication & Correlation |
| Dashboard-2 | Deep RDP Monitoring |
| Dashboard-3 | Privilege Escalation |
| Dashboard-4 | Persistence & Lateral Movement |
| Dashboard-5 | Threat Intelligence & IOC |

---

## Key Event IDs Used

| Event ID | Purpose |
|----------|---------|
| 4624 | Successful Logon |
| 4625 | Failed Logon |
| 4672 | Special Privileges Assigned |
| 4688 | Process Creation |
| 4697 | Service Installed |
| 4698 | Scheduled Task Created |
| 4728 | Added to Global Group |
| 4732 | Added to Local Group |
| 4756 | Added to Universal Group |
| 7045 | Service Creation |

---

## Operational Threshold Examples

| Condition | Severity |
|------------|----------|
| 5 min > 50 failed logins | High |
| Single IP > 15 fails in 5 min | High |
| Fail → Success same IP | High |
| External RDP success | Critical |
| Admin group membership change | Critical |
| Service creation + admin context | Critical |

---

## Notes

- Exclude machine accounts (`*$`) where applicable.
- If your field names differ (Windows event channel vs Sysmon vs custom decoders), adjust the KQL field paths accordingly.
- Prefer 5m histograms for spike detection dashboards.

---

Internal SOC Use Only

⚠ This blueprint is provided as a reference architecture. Adapt to your own environment before production deployment.
