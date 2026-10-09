# Agent Operational Directives — Wazuh SOC Blueprint

## Core Behavioral Protocols
1. **No Fluff**: Do not include conversational greetings, apologies, or boilerplate pleasantries. Focus directly on code, DQL queries, configurations, and analytical reports.
2. **Anti-Eager Execution**: When proposing a multi-step plan, halt tool execution immediately after presenting the plan to wait for human confirmation. Never execute file modifications without explicit permission.
3. **Evidence Over Assumption**: Never guess Wazuh decoders, Windows Event IDs, or OpenSearch field paths. Reference `.specs/boundary-conditions.md` and official Wazuh documentation.

## Project-Specific Rules
1. **Query Language**: Exclusively use **DQL**. Never output `KQL` or `KQL:`.
2. **Panel Formatting**: Every dashboard panel MUST implement all mandatory sections specified in `.specs/constitution.md` (Purpose, DQL, Visualization, Metrics, Buckets, SOC notes, Separator).
3. **Overlap Enforcement**: Before writing any new panel, check `.specs/boundary-conditions.md` to prevent duplicate or conflicting queries.
4. **Machine Account Filter**: User-centric detection queries must always exclude machine accounts:
   ```dql
   AND NOT data.win.eventdata.targetUserName:*$
   ```
5. **Local Rules Namespace**: Custom rules in `rules/local_rules.xml` must strictly utilize rule IDs in the `100000+` range and include valid MITRE ATT&CK tags.
