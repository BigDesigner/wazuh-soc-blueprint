# ADR 0001: Wazuh 4.x and OpenSearch Dashboards Technology Stack

- **Status**: Accepted
- **Confidence**: Verified
- **Date**: 2026-05-02

## Context
Security Operations Centers require a unified, cost-effective, and open architecture for security monitoring, log aggregation, and threat detection. Proprietary SIEM platforms often introduce high ingestion licensing costs, while ad-hoc log servers lack unified agent telemetry and active response capabilities.

## Decision
We adopt **Wazuh 4.x** paired with **Wazuh Indexer (OpenSearch)** and **Wazuh Dashboards (OpenSearch Dashboards)** as the core SIEM foundation:
1. Endpoints stream events via lightweight Wazuh Agents.
2. High-fidelity endpoint logs are collected through Windows Advanced Audit Policies and Microsoft Sysmon.
3. Dashboards, visualizations, and alerts are hosted natively on OpenSearch Dashboards using index pattern `wazuh-alerts-*`.

## Consequences
- Requires strict adherence to OpenSearch Dashboards query paradigms.
- Eliminates dependency on proprietary ingestion licenses.
- Preserves compatibility with OpenSearch Index State Management (ISM) and alerting integrations.

## Evidence
- `README.md` (Architecture Assumptions: Wazuh Manager 4.x, OpenSearch, index pattern `wazuh-alerts-*`).
- `rules/local_rules.xml` (Wazuh custom XML schema).
- `ilm/Index-Lifecycle-Policy.md` (OpenSearch ISM JSON policies).
