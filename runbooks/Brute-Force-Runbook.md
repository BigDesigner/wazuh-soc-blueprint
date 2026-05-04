# 🔨 Brute Force & Password Spraying Runbook — Wazuh SOC

**Version:** 1.0  
**Status:** Production Ready  
**Scope:** Response to automated authentication attacks detected via Dashboard-1.

---

## 🔍 1. Detection Signals

Identify the attack type based on Dashboard-1 indicators:
- **Brute Force:** High frequency of failures against a **single** account from a **single** IP.
- **Password Spraying:** Low frequency of failures against **multiple** accounts from a **single** IP.
- **Credential Stuffing:** High frequency of failures using **multiple** accounts and **multiple** IPs.

---

## 🚦 2. Triage Steps (Use Dashboard-1)

1. **Top Source IPs:** Check Panel 2 (Top Source IP) to identify the aggressor.
2. **Targeted Accounts:** Check Panel 3 (Top Target Users) to see which accounts are under fire.
3. **Success Correlation (CRITICAL):**
   - Check Panel 6 (Failed vs Success Correlation).
   - **Search:** `data.win.eventdata.ipAddress: <ATTACKER_IP> AND rule.groups: authentication_success`
   - If any success is found from the attacker IP → **Escalate to High/Critical immediately.**

---

## 🛡️ 3. Response Actions

### 3.1 Immediate Containment
- **Source IP Blocking:** If the IP is external and not a known VPN/Office IP, block it at the perimeter firewall.
- **Wazuh Active Response:** Trigger the `host-deny` or `firewall-drop` script if configured.
- **Account Lockout:** If multiple successes occurred, temporarily disable the affected accounts.

### 3.2 Eradication & Recovery
- **Password Reset:** Force a password change for all accounts that showed a "Success" event following failures from the same IP.
- **MFA Review:** Verify if MFA was bypassed or if the account lacks MFA protection.

---

## 📝 4. Post-Incident Tuning

- **Threshold Adjustment:** If the alert was a false positive (e.g., a service account with an expired password), update `local_rules.xml` or add the account to the exclusion list in the DQL.
- **Baseline Update:** Update the "Known Good" IP list in the SOC documentation.

---

END OF BRUTE FORCE RUNBOOK
