# Bootstrap & Deployment Blueprint

**Scope:** Wazuh SOC Blueprint Setup & Automation  
**Standard Version:** 1.1.0  
**Status:** Verified  

---

## 1. System Prerequisites

- **Wazuh Manager**: Version 4.x (Active and healthy)
- **Wazuh Indexer**: OpenSearch cluster status `Green`
- **Wazuh Dashboards**: Configured with default index pattern: `wazuh-alerts-*`
- **Endpoints**:
  - Windows workstations and servers with Advanced Audit Policy configured via GPO.
  - Microsoft Sysmon installed (SwiftOnSecurity or Olaf Hartong configuration recommended).
  - Wazuh Agent 4.x deployed and communicating.

---

## 2. Wazuh Rules Deployment (`local_rules.xml`)

1. **File Location**: Copy custom rules from `rules/local_rules.xml` to the Wazuh Manager:
   ```bash
   cp rules/local_rules.xml /var/ossec/etc/rules/local_rules.xml
   ```
2. **Permission Check**:
   ```bash
   chown wazuh:wazuh /var/ossec/etc/rules/local_rules.xml
   chmod 660 /var/ossec/etc/rules/local_rules.xml
   ```
3. **Rule Syntax Validation**:
   ```bash
   /var/ossec/bin/wazuh-analysisd -t
   ```
   *Expected output:* Valid configuration without syntax errors.
4. **Restart Wazuh Manager**:
   ```bash
   systemctl restart wazuh-manager
   ```

---

## 3. OpenSearch Dashboards Deployment

### Method A: Manual Creation (Recommended for Initial Customization)
1. Navigate to **OpenSearch Dashboards > Visualizations > Create New**.
2. Select visualization type specified in the markdown blueprint (e.g., Data Table, Horizontal Bar, Line Chart).
3. Select index pattern: `wazuh-alerts-*`.
4. Paste the exact DQL query from the `**DQL**` code fence.
5. Configure **Metrics** (Aggregation, Field, Custom Label) per specification.
6. Configure **Buckets** (Terms, Date Histogram, Split rows) per specification.
7. Save visualization and assemble onto the corresponding SOC Dashboard.

### Method B: Infrastructure-as-Code (NDJSON Import/Export)
To automate deployment across staging and production clusters:

#### 1. NDJSON Object Structure
A standard OpenSearch export object adheres to:
```json
{
  "id": "soc-dashboard-1",
  "type": "dashboard",
  "attributes": {
    "title": "SOC Dashboard-1 — Authentication & Correlation",
    "description": "Production blueprint for authentication monitoring",
    "panelsJSON": "[...]",
    "optionsJSON": "{\"useMargins\":true,\"hidePanelTitles\":false}",
    "version": 1
  },
  "references": []
}
```

#### 2. Export Workflow
1. Build and verify the dashboard in a staging OpenSearch cluster.
2. Go to **Stack Management > Saved Objects**.
3. Filter by Dashboard / Visualizations, click **Export**, and toggle "Include related objects".
4. Save exported `.ndjson` files to `dashboards/exports/`.

#### 3. Import Workflow
1. Go to **Stack Management > Saved Objects > Import**.
2. Upload the target `.ndjson` file.
3. Automatically maps to `wazuh-alerts-*` index pattern.

---

## 4. Index Lifecycle Management (OpenSearch ISM)

Apply the Index State Management policy via the OpenSearch REST API:

```bash
curl -k -u <ADMIN_USER>:<ADMIN_PASSWORD> -X PUT "https://<OPENSEARCH_IP>:9200/_plugins/_ism/policies/wazuh-alerts-lifecycle-policy" \
  -H "Content-Type: application/json" \
  -d @ilm/Index-Lifecycle-Policy.json
```

---

## 5. Slack Webhook Alerting Integration

Add the integration block to `/var/ossec/etc/ossec.conf` on the Wazuh Manager:

```xml
<ossec_config>
  <integration>
    <name>slack</name>
    <hook_url>https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK</hook_url>
    <level>10</level>
    <group>authentication_failed,threat_intel,persistence</group>
    <alert_format>json</alert_format>
  </integration>
</ossec_config>
```

Verify permissions and restart `wazuh-manager`.
