# Database Schema Documentation

| Field | Value |
|-------|-------|
| **System** | [System Name] |
| **Database** | [PostgreSQL/MySQL/etc.] |
| **Last Updated** | [DD Month YYYY] |

---

## 1. Schema Overview
[ER diagram or link to ERD document.]

## 2. Tables
### [table_name]
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK, NOT NULL | |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | |

## 3. Migration History
| Version | Description | Date | Author |
|---------|-------------|------|--------|
| 001 | Initial schema | | |

## 4. Migration Commands
```bash
# Run migrations
[command]

# Create new migration
[command]

# Rollback last migration
[command]
```

## 5. Seed Data
[Describe initial/required seed data.]

## 6. Backup & Restore
```bash
# Backup
[command]

# Restore
[command]
```
