# Configuration Management

| Field | Value |
|-------|-------|
| **System** | [System Name] |
| **Last Updated** | [DD Month YYYY] |

---

## 1. Environments
| Environment | URL | Purpose |
|-------------|-----|---------|
| Local | localhost:XXXX | Development |
| Dev | dev.example.com | Integration testing |
| Staging | staging.example.com | Pre-production |
| Production | example.com | Live |

## 2. Environment Variables
| Variable | Local | Dev | Staging | Prod | Description |
|----------|-------|-----|---------|------|-------------|
| | | | | | |

## 3. Feature Flags
| Flag | Local | Dev | Staging | Prod | Description |
|------|-------|-----|---------|------|-------------|
| | on/off | | | | |

## 4. Secrets Management
- **Tool:** [Vault/AWS Secrets Manager/Azure Key Vault]
- **Access:** [How to access secrets]
- **Rotation:** [Rotation policy]

## 5. Adding New Configuration
1. Add to `.env.example` with description
2. Update this document
3. Add to deployment config
4. Notify team
