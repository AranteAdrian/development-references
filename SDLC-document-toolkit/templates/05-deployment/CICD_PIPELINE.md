# CI/CD Pipeline Documentation

| Field | Value |
|-------|-------|
| **System** | [System Name] |
| **Tool** | [GitHub Actions / Azure DevOps / Jenkins] |
| **Last Updated** | [DD Month YYYY] |

---

## 1. Pipeline Architecture
[Diagram or description of pipeline stages.]

## 2. Stages
| Stage | Trigger | Actions | Duration |
|-------|---------|---------|----------|
| Build | Push to any branch | Compile, lint | ~2 min |
| Test | Push to any branch | Unit + integration tests | ~5 min |
| Security | Push to main/develop | SAST, dependency scan | ~3 min |
| Deploy (Dev) | Merge to develop | Auto-deploy | ~3 min |
| Deploy (Staging) | Merge to main | Auto-deploy | ~3 min |
| Deploy (Prod) | Manual approval | Deploy + smoke test | ~5 min |

## 3. Quality Gates
| Gate | Criteria | Blocking? |
|------|---------|-----------|
| Tests | 100% pass | Yes |
| Coverage | >= [X]% | Yes |
| Lint | 0 errors | Yes |
| Security | 0 critical/high | Yes |

## 4. Environment Promotion
```
feature branch → develop → staging → production
                  (auto)    (auto)    (manual gate)
```

## 5. Secrets Management
[How secrets are injected into the pipeline.]
