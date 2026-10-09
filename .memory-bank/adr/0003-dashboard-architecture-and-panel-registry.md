# ADR 0003: SOC Dashboard Architecture & Panel Registry

- **Status**: Accepted
- **Confidence**: Verified
- **Date**: 2026-05-04

## Context
As SOC monitoring grows, multiple dashboards can easily suffer from redundant queries, duplicate panels, inconsistent triage notes, and unclear severity boundaries. An unorganized dashboard set introduces alert fatigue, wastes analyst time, and creates maintenance overhead.

## Decision
We establish a modular 6-dashboard architecture governed by a central panel registry and strict overlap detection rules:
1. **Dashboard-1**: Authentication & Correlation (7 panels)
2. **Dashboard-2**: Deep RDP Monitoring & VPN Correlation (8 panels)
3. **Dashboard-3**: Privilege Escalation & Admin Account Manipulation (4 panels)
4. **Dashboard-4**: Persistence & Lateral Movement (5 panels)
5. **Dashboard-5**: Threat Intelligence & IOC Monitoring (5 panels)
6. **Dashboard-6**: Execution, Injection & Credential Dumping (5 panels)

Each panel must register its primary event ID, rule group, visualization type, and bucket field in `.specs/boundary-conditions.md`. Any new panel proposal must execute the 3-step overlap detection algorithm prior to inclusion.

## Consequences
- Guarantees orthogonal, complementary coverage across the 29 baseline panels.
- Enables clear operational handoffs between Tier 1 triage and Tier 2 deep investigations.
- Ensures all MITRE ATT&CK techniques map cleanly to specific panels without ambiguous duplicates.

## Evidence
- `.specs/boundary-conditions.md` (Master Panel Registry and Overlap Governance).
- `mitre/MITRE-Coverage-Matrix.md` (Technique mapping table).
- Commits `1605b0b` and `63ada8c`.
