# 📢 Slack Alert Integration Guide — Wazuh SOC

**Version:** 1.0  
**Scope:** Automated alerting via Wazuh Integratord daemon.

---

## 🛠️ 1. Wazuh Manager Configuration (`ossec.conf`)

Add the following block to your Wazuh manager's configuration file. This setup uses a custom integration script to format the JSON payload for Slack.

```xml
<ossec_config>
  <integration>
    <name>slack</name>
    <hook_url>https://hooks.slack.com/services/YOUR_WORK_SPACE/YOUR_BOT/YOUR_SECRET_TOKEN</hook_url> <!-- Replace with your Webhook URL -->
    <level>10</level> <!-- Alert on level 10 and above -->
    <alert_format>json</alert_format>
  </integration>
</ossec_config>
```

---

## 🎨 2. Recommended Slack Payload Template

The following JSON structure is recommended for the Slack message to ensure SOC analysts have the necessary context at a glance.

```json
{
  "attachments": [
    {
      "fallback": "Wazuh Alert: ${rule.description}",
      "color": "#FF0000",
      "pretext": "🚨 *Wazuh Security Alert*",
      "title": "Rule: ${rule.description}",
      "title_link": "https://wazuh-dashboard.internal/app/discover#/",
      "fields": [
        {
          "title": "Severity",
          "value": "Level ${rule.level}",
          "short": true
        },
        {
          "title": "Agent Name",
          "value": "${agent.name}",
          "short": true
        },
        {
          "title": "Source IP",
          "value": "${data.srcip}",
          "short": true
        },
        {
          "title": "MITRE Technique",
          "value": "${rule.mitre.id} (${rule.mitre.tactic})",
          "short": false
        }
      ],
      "footer": "Wazuh SOC Blueprint",
      "ts": "${timestamp}"
    }
  ]
}
```

---

## 🚦 3. Severity-Based Routing Logic

| Wazuh Level | Slack Color | Action |
|-------------|-------------|--------|
| **Level 3-7** | `#FFFF00` (Yellow) | Informational / Batch channel |
| **Level 8-11** | `#FFA500` (Orange) | SOC Main channel |
| **Level 12-15** | `#FF0000` (Red) | SOC Critical / PagerDuty channel |

---

## 💡 SOC Notes for Analysts

- **Deep Link:** Always include a link back to the Wazuh Dashboard filtered by the `alert.id` or `agent.id`.
- **Throttling:** Use Wazuh's `alert_format` or external logic to prevent alert fatigue during brute-force attacks (already mitigated by the frequency setting in `local_rules.xml`).
- **Context:** Ensure `rule.groups` are included in the Slack metadata for better searchability within Slack history.

---

END OF SLACK INTEGRATION TEMPLATE
