# Brute Force Runbook (Template)

## Signals
- Spike in 4625 / authentication_failed
- Single IP concentrated attempts
- Multiple users targeted (spray)

## Immediate Actions
- Identify top source IPs and targeted accounts
- Confirm if any success occurred after failures
- Block/contain at perimeter (per policy)
- Force password reset if compromise suspected

## Post Actions
- Add allowlists / suppressions for known scanners
- Tune thresholds per environment baseline
