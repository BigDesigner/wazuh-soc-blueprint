# Migration Map — Sentinel Memory Bank Initialization

This document tracks the migration of legacy documentation and specification files into the Sentinel Agent Memory Bank structure.

> [!NOTE]
> Per explicit user instruction, legacy documentation files are migrated completely and losslessly into `.specs/`, `.memory-bank/`, `.agents/`, and `.tasks/`, and deleted permanently without creating an `.archive/` directory.

| Original Path | New Target Path | Archive Path | Action | Notes / Preserved Content |
|---|---|---|---|---|
| `docs/AI-Agent-Panel-Standard.md` | `.specs/constitution.md`, `.agents/AGENTS.md`, `.memory-bank/adr/0002-dql-query-standard.md` | None (Deleted per user instruction) | Migrated | Full preservation of Rules 0–12, panel structure, DQL syntax rules, metric aggregations, bucket schemas, and copy-paste templates. |
| `docs/AI-Agent-Overlap-Rules.md` | `.specs/boundary-conditions.md`, `.memory-bank/adr/0003-dashboard-architecture-and-panel-registry.md` | None (Deleted per user instruction) | Migrated | Full preservation of overlap definition matrix, 3-step detection algorithm, warning templates, and the complete 29-panel registry table. |
| `docs/Project-Analysis.md` | `.memory-bank/audits/audit-initial-analysis.md`, `.memory-bank/bugs/bug-list.md`, `.tasks/pipeline.md`, `.memory-bank/changelog/verified-worklog.md` | None (Deleted per user instruction) | Migrated | Full preservation of maturity matrix, strengths, findings BULGU-001 through BULGU-012, MITRE gaps, and roadmap priorities. |
| `docs/Dashboard-4-5-Rewrite-Roadmap.md` | `.memory-bank/changelog/verified-worklog.md`, `.tasks/pipeline.md` | None (Deleted per user instruction) | Migrated | Full preservation of D4 and D5 rewrite specifications and completion status. |
| `docs/OpenSearch-Import-Guide.md` | `.specs/bootstrap.md`, `.memory-bank/adr/0001-wazuh-opensearch-stack.md` | None (Deleted per user instruction) | Migrated | Full preservation of manual setup instructions, NDJSON schema/export strategy, and index pattern configurations. |
