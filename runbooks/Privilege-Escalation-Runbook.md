# 👑 Privilege Escalation Runbook — Wazuh SOC

**Version:** 1.0  
**Status:** Production Ready  
**Scope:** Response to unauthorized privilege elevation or administrative group changes detected via Dashboard-3.

---

## 🔍 1. Detection Signals

Monitor Dashboard-3 for the following critical indicators:
- **Admin Group Changes:** Event IDs 4728, 4732, 4756 (User added to security-enabled groups).
- **Special Privileges:** Event ID 4672 (Assigned to new logon sessions).
- **Sensitive Account Creation:** Event ID 4720 (New user accounts, especially those added to groups).
- **Service-Based Escalation:** Event ID 7045 (New service creation, often running as SYSTEM).

---

## 🚦 2. Triage & Verification (Use Dashboard-3)

1. **Identify the Actor:** Check Panel 1 (Special Privileges) or Panel 2 (Admin Group Changes) for the `subjectUserName` (the one who made the change).
2. **Identify the Target:** Check `targetUserName` or `memberName` to see who received the privileges.
3. **Validation Check:**
   - Cross-reference with the **IT Change Management System** (e.g., Jira, ServiceNow).
   - If no valid ticket exists → **Treat as a CRITICAL security incident.**
4. **Context Analysis:** Check Dashboard-6 (Execution) to see what the actor did immediately before and after the privilege change (e.g., suspicious process execution).

---

## 🛡️ 3. Response Actions

### 3.1 Immediate Containment
- **Unauthorized Change Reversal:** Immediately remove the target user from the privileged group.
- **Actor Lockdown:** Disable the `subjectUserName` account that initiated the unauthorized change.
- **Host Isolation:** If the change originated from a non-IT management host, isolate that agent via Wazuh Active Response.

### 3.2 Eradication & Recovery
- **Audit:** Review all actions performed by the target user after receiving privileges.
- **Cleanup:** Delete any unauthorized services (7045) or scheduled tasks (4698) created by the user.

---

## 📝 4. Post-Incident Tuning

- **Reporting:** Document the `mitre.id` (e.g., T1078.002, T1548) for the incident report.
- **Architecture Review:** Evaluate the need for Just-In-Time (JIT) access or LAPS (Local Administrator Password Solution) to reduce the attack surface.

---

END OF PRIVILEGE ESCALATION RUNBOOK
