# Project Constitution & Engineering Standards

**Scope:** Wazuh SOC Blueprint  
**Standard Version:** 1.1.0  
**Status:** Enforced  

---

## 1. Core Principles (Rule 0)

1. **DQL Exclusivity**: Query language MUST strictly be Dashboards Query Language (`DQL`). The label `KQL` is strictly forbidden. Wazuh Indexer (OpenSearch) does not support KQL syntax natively.
2. **Single Detection Intent**: Each panel must serve exactly one detection or triage objective. Multi-purpose or ambiguous panels are prohibited.
3. **Copy-Paste Readiness**: Every panel specification must contain sufficient detail (DQL, metrics, aggregations, bucket paths) so an analyst or automated script can implement it directly in OpenSearch Dashboards without guessing.
4. **Zero Duplication**: No panel may duplicate the identical `DQL + Visualization + Bucket` combination of an existing panel across the repository.

---

## 2. Dashboard Header Standard (Rule 1)

Every dashboard markdown template must open with the following exact header structure:

```markdown
# SOC Dashboard-{N} — {Title} (Production Template)

**Index pattern:** `wazuh-alerts-*`  
**Time range:** `Last 24 hours` (default; override per investigation)  
**Query language:** **DQL**

> Dashboard-{N} purpose: {Single sentence summarizing the detection objective}.

---
```

### Header Constraints
- Title MUST use H1 `# SOC Dashboard-{N} — {Title} (Production Template)`.
- Index pattern is permanently set to `wazuh-alerts-*`.
- Query language MUST be labeled `**DQL**`.
- Purpose blockquote MUST provide a concise, high-level SOC rationale.

---

## 3. Mandatory Panel Section Hierarchy (Rule 2)

Every panel within a dashboard must follow this sequential structure:

```markdown
## Panel {N} — SOC | {Category} | {Name}

**Purpose:** {Single sentence explaining what is detected and why it matters}.

**DQL**
```dql
{query}
```

**Visualization:** {Type}

**Metrics**
- Aggregation: `{Count | Unique Count | Sum | Average}`
- Field: `{field_path}` (Required when aggregation is not Count)
- Custom Label: `{Descriptive Label}`

**Buckets** or **Buckets (Split rows order)**
{Bucket specifications}

**SOC notes**
- {Observation 1} → {Operational action/implication}.
- {Observation 2} → {Operational action/implication}.
- Correlate with: {Related dashboard panel or Windows Event ID}.
- Severity: **{Severity Level}**

---
```

---

## 4. Panel Naming & Hierarchy (Rule 3)

- **Level**: Must use H2 (`##`). Never use H1 (`#`).
- **Format**: `## Panel {N} — SOC | {Category} | {Name}`
- **Prefix**: `SOC |` is mandatory.
- **Allowed Categories**: `KPI`, `Top`, `Spike`, `Correlation`, `RDP`, `Auth`, `Persistence`, `Privilege`, `TI`, `Execution`, `Defense Evasion`.

---

## 5. Purpose & Detection Intent (Rule 4)

- Must use `**Purpose:** {Statement}` format.
- Exactly one clear sentence defining detection scope and relevance.
- Must cite relevant Event IDs or logon types where applicable.

---

## 6. DQL Query Formatting (Rule 5)

- Code fence tag MUST be ` ```dql `.
- **Machine Account Exclusion**: For all user-facing authentication and execution queries, always append:
  ```dql
  AND NOT data.win.eventdata.targetUserName:*$
  ```
- **Loopback & Noise Suppression**: For IP-based queries, suppress loopbacks and IPv6 link-locals:
  ```dql
  AND NOT data.win.eventdata.ipAddress:("::1")
  AND NOT data.win.eventdata.ipAddress:(fe80* OR 2001*)
  ```
- **Multi-line Formatting**: Complex boolean expressions must place `AND` / `OR` on new lines with clear indentation.

---

## 7. Visualization Types & Metrics (Rules 6 & 7)

### Permitted Visualizations
| Visualization | Typical Use Case | Buckets Required |
|---|---|---|
| `Metric` | KPI single numeric counters | No |
| `Horizontal Bar` | Top-N ranking (IPs, accounts, domains) | Yes |
| `Vertical Bar` | Time or categorical comparisons | Yes |
| `Line Chart` | Time-series trends and spike detection (Date Histogram) | Yes |
| `Bar Chart` | Multi-dimension / split-series timelines | Yes |
| `Data Table` | Detailed investigative drill-downs (multi-level aggregation) | Yes |
| `Pie` | Proportional distributions (MITRE, rule levels) | Yes |

### Metric Aggregations
- `Count`: Standard alert occurrences.
- `Unique Count`: Distinct entity counts (e.g. `Unique Count` on `agent.name`). Field is required.
- `Sum`, `Average`, `Max`, `Min`: Numeric field statistics.

---

## 8. Bucket Configurations (Rule 8)

### Variation A: Single Bucket (Bar / Pie)
```markdown
**Buckets**
- Aggregation: `Terms`
- Field: `{field_path}`
- Order by: `Count`
- Order: `Descending`
- Size: `{N}`
- Custom Label: `{Label}`
```

### Variation B: Multi-Bucket / Split Rows (Data Table)
```markdown
**Buckets (Split rows order)**
1) Aggregation: `Terms`
   - Field: `{field_1}`
   - Order by: `Count`
   - Order: `Descending`
   - Size: `{N}`
   - Custom Label: `{Label 1}`

2) Sub Aggregation: `Terms`
   - Field: `{field_2}`
   - Order by: `Count`
   - Order: `Descending`
   - Size: `{N}`
   - Custom Label: `{Label 2}`
```

### Variation C: Timeline (Date Histogram + Split Series)
```markdown
**Buckets**
- X-axis:
  - Aggregation: `Date Histogram`
  - Field: `@timestamp`
  - Interval: `{5m | 1h | 3h}`
  - Custom Label: `Time ({interval})`
- Split series:
  - Aggregation: `Terms`
  - Field: `{field_path}`
  - Order by: `Count`
  - Order: `Descending`
  - Size: `{N}`
  - Custom Label: `{Label}`
```

---

## 9. SOC Notes & Severity Standards (Rule 9)

Every panel must provide actionable triage guidance for Tier 1 / Tier 2 SOC analysts.
- Use the arrow operator `→` connecting observation to triage implication.
- Specify cross-dashboard correlation pathways.
- Explicit severity declaration using bold formatting:
  - `**Low**`: Informational / baseline monitoring.
  - `**Medium**`: Anomalous activity requiring contextual verification.
  - `**Medium → High**`: Variable risk depending on asset criticality or frequency.
  - `**High**`: Strong indicator of compromise or unauthorized activity.
  - `**High → Critical**`: Dangerous operation on core assets (Domain Controllers, DBs).
  - `**Critical**`: Immediate escalation, host containment, and active incident response.

---

## 10. Dashboard Footer (Rule 10)

Every dashboard markdown file must conclude with:

```markdown
---

END OF DASHBOARD-{N} TEMPLATE
```

---

## 11. Strict Prohibitions & Anti-Patterns (Rule 12)

1. ❌ **No KQL**: Never write `KQL` or `KQL:` in any file.
2. ❌ **No H1 Panel Headers**: Panel headers must never use single `#`.
3. ❌ **No Missing SOC Notes**: Panels without triage notes or severity are invalid.
4. ❌ **No Missing Bucket Definitions**: Non-metric visualizations must explicitly specify aggregations.
5. ❌ **No Untracked Overlap**: Adding queries that duplicate existing coverage without registering in `.specs/boundary-conditions.md` is forbidden.
