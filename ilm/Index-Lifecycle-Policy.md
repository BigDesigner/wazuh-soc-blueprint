# Index Lifecycle Policy (Placeholder)

Define retention tiers (example):
- Hot: 7-14 days (high-performance)
- Warm: 30-90 days (cost-optimized)
- Delete: 90-180 days (compliance-dependent)

Apply to:
- wazuh-alerts-*

Ensure:
- Disk watermarks configured
- Shard count tuned to node resources
