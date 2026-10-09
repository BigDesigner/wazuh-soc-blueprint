# ADR 0002: Dashboards Query Language (DQL) Standardization

- **Status**: Accepted
- **Confidence**: Verified
- **Date**: 2026-05-02

## Context
Initial blueprints occasionally referenced Kibana Query Language (`KQL`). However, Wazuh Indexer is based on OpenSearch, which implements Dashboards Query Language (`DQL`). Attempting to use KQL constructs or labeling queries as KQL creates operational confusion and import incompatibilities in OpenSearch Dashboards.

## Decision
All dashboard queries, search bars, and visualization specifications must exclusively use **DQL**. The label `KQL` is strictly deprecated and prohibited across the repository. Queries must use standard boolean operators (`AND`, `OR`, `NOT`), wildcards (`*`), and field-value mappings compatible with OpenSearch Lucene/DQL parser.

## Consequences
- Every dashboard template (Dashboards 1–6) is standardized with `**DQL**` and ` ```dql ` syntax blocks.
- Analysts can directly copy and paste queries into the OpenSearch search bar without syntax conversion errors.
- Any future automated parser or validation tool will validate against DQL syntax specifications.

## Evidence
- `.specs/constitution.md` (Rule 0.1 and Rule 5 explicitly banning KQL).
- Commit `1605b0b` (Migration of Dashboard 4 and Dashboard 5 to DQL format).
- `dashboards/SOC-Dashboard-1.md` through `SOC-Dashboard-6-Execution-Process.md`.
