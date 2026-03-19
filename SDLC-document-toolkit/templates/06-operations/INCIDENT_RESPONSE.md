# Incident Response Plan

| Field | Value |
|-------|-------|
| **System** | [System Name] |
| **Owner** | [Name] |
| **Last Updated** | [DD Month YYYY] |

---

## 1. Severity Definitions
| Severity | Impact | Response Time | Examples |
|----------|--------|--------------|----------|
| SEV1 | Complete outage | 15 min | System down |
| SEV2 | Major degradation | 30 min | Feature broken |
| SEV3 | Minor impact | 4 hours | Slow performance |
| SEV4 | Minimal | Next business day | UI glitch |

## 2. Roles
| Role | Responsibility | Current Owner |
|------|---------------|--------------|
| Incident Commander | Coordinates response | |
| Communications Lead | Updates stakeholders | |
| Technical Lead | Diagnoses and fixes | |

## 3. Response Process
1. **Detect** — alert fires or user reports
2. **Triage** — assess severity
3. **Assemble** — page appropriate responders
4. **Investigate** — diagnose root cause
5. **Mitigate** — restore service (fix or workaround)
6. **Resolve** — permanent fix
7. **Review** — post-incident review

## 4. Communication Templates
### Internal (Slack)
```
🔴 SEV[X] — [System]: [Brief description]
IC: [Name] | Status: Investigating
Bridge: [link]
```

### External (Status Page)
```
We are investigating [issue]. Updates every [X] minutes.
```

## 5. Escalation Matrix
| Condition | Escalate To | Contact |
|-----------|------------|---------|
| SEV1 not resolved in 30 min | Engineering Director | |
| Data breach suspected | Security + Legal | |
