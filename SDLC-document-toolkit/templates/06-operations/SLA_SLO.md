# SLA / SLO Documentation

| Field | Value |
|-------|-------|
| **Service** | [Service Name] |
| **Owner** | [Name] |
| **Last Reviewed** | [DD Month YYYY] |

---

## 1. Service Level Indicators (SLIs)
| SLI | What It Measures | How It's Measured |
|-----|-----------------|------------------|
| Availability | % of successful requests | Success / Total requests |
| Latency | Response time at p99 | APM tool |
| Error rate | % of 5xx responses | Monitoring |

## 2. Service Level Objectives (SLOs)
| SLI | Objective | Measurement Window |
|-----|-----------|-------------------|
| Availability | >= 99.9% | 30-day rolling |
| Latency (p99) | < 500ms | 30-day rolling |
| Error rate | < 0.1% | 30-day rolling |

## 3. Error Budget
| SLO | Budget (30 days) | Current Remaining |
|-----|-----------------|-------------------|
| 99.9% availability | 43.2 min downtime | [X] min |

## 4. SLA (External Commitments)
| Metric | Commitment | Penalty |
|--------|-----------|---------|
| Uptime | 99.9% monthly | [credit/refund terms] |

## 5. Review Cadence
- **Weekly:** Check error budget burn rate
- **Monthly:** SLO review with team
- **Quarterly:** SLA review with stakeholders
