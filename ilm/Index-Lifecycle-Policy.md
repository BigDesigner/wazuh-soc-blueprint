# 💾 Index Lifecycle & ISM Policy — Wazuh SOC

**Version:** 1.0  
**Target:** OpenSearch Dashboards / Wazuh Indexer  
**Scope:** `wazuh-alerts-*` index pattern retention management.

---

## 🏗️ 1. OpenSearch ISM Policy (JSON)

Copy and paste this JSON into **OpenSearch Dashboards > Index Management > State Management Policies**.

```json
{
  "policy": {
    "description": "Wazuh SOC Blueprint - 90 Days Retention Policy (Hot-Warm-Cold-Delete)",
    "default_state": "hot",
    "states": [
      {
        "name": "hot",
        "actions": [
          {
            "rollover": {
              "min_index_age": "24h",
              "min_primary_shard_size": "30gb"
            }
          }
        ],
        "transitions": [
          {
            "state_name": "warm",
            "conditions": {
              "min_index_age": "7d"
            }
          }
        ]
      },
      {
        "name": "warm",
        "actions": [
          {
            "replica_count": {
              "number_of_replicas": 1
            }
          }
        ],
        "transitions": [
          {
            "state_name": "cold",
            "conditions": {
              "min_index_age": "30d"
            }
          }
        ]
      },
      {
        "name": "cold",
        "actions": [
          {
            "read_only": {}
          }
        ],
        "transitions": [
          {
            "state_name": "delete",
            "conditions": {
              "min_index_age": "90d"
            }
          }
        ]
      },
      {
        "name": "delete",
        "actions": [
          {
            "delete": {}
          }
        ],
        "transitions": []
      }
    ],
    "ism_template": [
      {
        "index_patterns": ["wazuh-alerts-4.x-*"],
        "priority": 100
      }
    ]
  }
}
```

---

## 📊 2. Tier Definitions

| State | Duration | Action | Purpose |
|-------|----------|--------|---------|
| **Hot** | 0-7 Days | Rollover @ 30GB or 24h | Active search & high-speed indexing. |
| **Warm** | 7-30 Days | Set 1 Replica | Optimized for search performance vs cost. |
| **Cold** | 30-90 Days | Read-Only | Long-term archival for incident lookback. |
| **Delete** | 90+ Days | Permanent Delete | Compliance cleanup (adjust based on policy). |

---

## 🚦 3. Implementation Steps

1. **Verify Index Pattern:** Ensure your Wazuh indices match `wazuh-alerts-4.x-*` or adjust the `ism_template`.
2. **Disk Check:** Monitor the "High Watermark" (default 90%) to prevent indices from locking.
3. **Shard Tuning:** Maintain shard sizes between 30GB and 50GB for optimal performance.

---

END OF ISM POLICY TEMPLATE
