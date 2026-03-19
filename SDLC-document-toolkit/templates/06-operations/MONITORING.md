# Monitoring & Alerting Configuration

| Field | Value |
|-------|-------|
| **System** | [System Name] |
| **Tools** | [Datadog/Prometheus/Azure Monitor] |
| **Last Updated** | [DD Month YYYY] |

---

## 1. Dashboards
| Dashboard | URL | Purpose |
|-----------|-----|---------|
| Overview | [URL] | System health at a glance |
| API Performance | [URL] | Endpoint latency and errors |

## 2. Alert Rules
| Alert | Condition | Severity | Channel | Runbook |
|-------|-----------|----------|---------|---------|
| High error rate | 5xx > 5% for 5 min | SEV2 | PagerDuty | [link] |
| High latency | p99 > 2s for 10 min | SEV3 | Slack | [link] |

## 3. On-Call Rotation
| Week | Primary | Secondary |
|------|---------|-----------|
| | | |

## 4. Log Aggregation
- **Tool:** [ELK/Datadog/CloudWatch]
- **Retention:** [X] days
- **Key queries:** [list useful log queries]
