# System Coherence & Operational Governance

## 1. System Identity & Mission
The **Wazuh SOC Blueprint** is an enterprise-grade detection engineering and Security Operations Center (SOC) dashboard framework designed for Wazuh 4.x and OpenSearch Dashboards (Wazuh Indexer). Its primary objective is providing copy-paste ready, standardized detection dashboards, custom threshold rules, incident response runbooks, index lifecycle policies, and MITRE ATT&CK coverage maps.

## 2. Session Start Protocol & Operating Modes
- **Session Restoration**: On every session start, AI agents must read `.memory-bank/active-session.json`, `.tasks/pipeline.md`, and `.tasks/handoff.md`.
- **Locking Protocol**: Check `.memory-bank/.session.lock`. If present and younger than 10 minutes, halt and wait. If older than 10 minutes, log a warning in `.memory-bank/bugs/bug-list.md` and release the stale lock.
- **Atomic Writes**: Any update to `.memory-bank/active-session.json` MUST be performed via `active-session.tmp.json` before atomic rename.
- **Operating Modes**:
  - `Interactive Mode` (Default): Requires user approval before irreversible mutations, code restructuring, or git commits.
  - `CI Mode` (`CI=true`): Automated non-blocking mode; creates ADR proposals with `Unconfirmed` status where human review is required.

## 3. Worktree Cleanliness & Branch Awareness
- Always verify git status before making architectural alterations.
- Never stage or commit files without explicit user consent.
- All branches track `main` by default unless feature-isolated worktrees are designated.

## 4. Architectural Drift Prevention & Core Invariants
1. **DQL Exclusivity**: All OpenSearch / Wazuh Dashboards queries must use Dashboards Query Language (`DQL`). KQL is strictly banned because the OpenSearch engine does not parse KQL syntax correctly in Wazuh distributions.
2. **Standardized Panel Layout**: Every SOC panel must follow the strict 8-point section hierarchy defined in `.specs/constitution.md` (Purpose, DQL, Visualization, Metrics, Buckets, SOC notes, Separator).
3. **No Overlap Principle**: New detection panels must be cross-checked against `.specs/boundary-conditions.md` to prevent duplicate or conflicting queries.
4. **Noise Filtration**: Every user-centric authentication or process query must enforce machine account exclusion (`AND NOT data.win.eventdata.targetUserName:*$`) and local/loopback noise elimination.
5. **Lossless Memory Preservation**: Project decisions, rule thresholds, and operational runbooks must remain synchronized across `.specs/`, `.memory-bank/`, and actual dashboard markdown files.

## 5. Pre-Change & Post-Change Verification Checklist
- **Pre-Change**:
  1. Inspect existing panel registry in `.specs/boundary-conditions.md`.
  2. Verify Event ID mapping against Windows Security / Sysmon documentation.
  3. Ensure no logical or exact overlap with existing dashboards 1–6.
- **Post-Change**:
  1. Verify DQL syntax compliance.
  2. Verify MITRE ATT&CK technique IDs against `mitre/MITRE-Coverage-Matrix.md`.
  3. Update `.tasks/pipeline.md` and `.memory-bank/changelog/verified-worklog.md`.
  4. Ensure `.memory-bank/active-session.json` reflects the current sprint state.

## 6. Handoff Protocol
When concluding an operational session, generate or update `.tasks/handoff.md` detailing:
- Mode, active branch, and last commit hash.
- Summary of modifications, verifications performed, and outstanding backlog items.
- Recommended next steps for subsequent agents or human analysts.
