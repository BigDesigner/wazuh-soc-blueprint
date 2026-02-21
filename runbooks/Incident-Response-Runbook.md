# Incident Response Runbook (Template)

## Triage Order (Strict)
1. Service status / agent health
2. Scope (hosts/users/IPs)
3. Timeline (spike vs steady-state)
4. Credential impact (fail→success, 4672)
5. Lateral indicators (Type 3, 7045, remote task)
6. Persistence artifacts (4698, Run keys)
7. Containment actions (per policy)
8. Eradication & recovery
9. Lessons learned / tuning

## Evidence Checklist
- Affected hosts (agent.name)
- src.ip / dest.ip
- targetUserName / subjectUserName
- logonType
- process tree (4688 / Sysmon)
- IOC matches (Dashboard-5)

## Escalation
- Admin group change: Critical, immediate escalation
- External RDP success: Critical, immediate escalation
