# Privilege Escalation Runbook (Template)

## Signals
- 4728/4732/4756 group membership changes
- 4672 special privileges assigned
- 4697 service installed
- 7045 service created
- Abnormal admin process creation (4688)

## Immediate Actions
- Identify actor (subjectUserName) and target (memberName/targetUserName)
- Validate change request / ticket
- Isolate host if unauthorized
- Collect evidence (event logs + process tree)

## Post Actions
- Tighten admin delegation
- Add detections for common bypass tools
- Consider LAPS / JIT admin access model
