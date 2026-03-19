# Rollback Plan

| Field | Value |
|-------|-------|
| **Release** | v[X.Y.Z] |
| **Date** | [DD Month YYYY] |
| **Decision Authority** | [Name] |

---

## 1. Rollback Trigger Criteria
- [ ] Error rate exceeds [X]%
- [ ] Response time exceeds [X]ms for [X] minutes
- [ ] Critical functionality broken
- [ ] Data integrity issue detected

## 2. Rollback Steps
| # | Step | Command / Action | Verify |
|---|------|-----------------|--------|
| 1 | Revert application | | |
| 2 | Revert database (if needed) | | |
| 3 | Clear caches | | |
| 4 | Verify rollback | | |

## 3. Communication
| Audience | Channel | Template |
|----------|---------|---------|
| Engineering | Slack | "Rolling back v[X] due to [reason]" |
| Stakeholders | Email | |
| Users | Status page | |

## 4. Post-Rollback
- [ ] Incident created
- [ ] Post-mortem scheduled
- [ ] Root cause investigation started
