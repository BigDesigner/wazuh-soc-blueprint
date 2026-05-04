# 📥 OpenSearch Dashboard Import Guide

This guide explains how to transition from the Markdown templates in this repository to functional dashboards in OpenSearch Dashboards.

## 1. Manual Creation (Recommended for Initial Setup)

Since field names and index patterns can vary between environments, manual creation ensures the highest compatibility.

1.  **Open OpenSearch Dashboards.**
2.  **Go to Visualizations > Create New.**
3.  **Choose the Type** specified in the template (e.g., Data Table, Horizontal Bar).
4.  **Copy the DQL** from the `**DQL**` code block in the markdown file.
5.  **Configure Metrics and Buckets** exactly as described in the `**Metrics**` and `**Buckets**` sections.
6.  **Add to Dashboard:** Save the visualization and add it to a new or existing dashboard.

## 2. Infrastructure-as-Code (NDJSON) Approach

To automate deployment, you can use the NDJSON format.

### Template Structure (`.ndjson`)

A standard OpenSearch export file looks like this:

```json
{"id":"v1","objects":[{"id":"dashboard-id","type":"dashboard","attributes":{"title":"SOC Dashboard-1","panelsJSON":"[...]"},"references":[]}]}
```

### How to Generate NDJSON

1.  **Build one dashboard manually** following Step 1.
2.  **Go to Stack Management > Saved Objects.**
3.  **Select the Dashboard** and click **Export**.
4.  **Save the .ndjson file** in the `dashboards/exports/` directory of this repo.
5.  This file can now be version-controlled and imported into other environments.

## 3. Best Practices

- **Field Mapping:** Ensure `win.eventdata.*` fields are mapped correctly in your index pattern.
- **Refresh Interval:** Set a reasonable refresh interval (e.g., 1 minute) for live SOC monitoring.
- **Time Range:** Use "Last 24 hours" as the default but allow analysts to override for investigations.
