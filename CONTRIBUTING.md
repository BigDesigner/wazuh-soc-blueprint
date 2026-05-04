# Contributing to Wazuh SOC Blueprint

Thank you for your interest in improving the Wazuh SOC Blueprint!

## How to Contribute

1.  **Dashboard Panels:** Ensure all new panels follow the [AI-Agent-Panel-Standard.md](docs/AI-Agent-Panel-Standard.md).
2.  **DQL Queries:** Use DQL (Dashboard Query Language). KQL is not supported in the target environment.
3.  **MITRE Mapping:** Every new detection should be mapped to a MITRE ATT&CK technique in the [MITRE-Coverage-Matrix.md](mitre/MITRE-Coverage-Matrix.md).
4.  **Runbooks:** When adding a new dashboard, consider adding a corresponding section or a new file in the `runbooks/` directory.

## Quality Standards

- No overlapping panels (check [AI-Agent-Overlap-Rules.md](docs/AI-Agent-Overlap-Rules.md)).
- Mandatory `SOC notes` and `Severity` for all panels.
- Prefer `Data Table` for detailed investigations and `Horizontal Bar` for Top-N analysis.

## Bug Reports & Feature Requests

Please use the repository's issue tracker to report bugs or suggest new dashboards (e.g., Linux/macOS specific ones).
