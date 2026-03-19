# Coding Standards

| Field | Value |
|-------|-------|
| **Team** | [Team Name] |
| **Language(s)** | [Python, TypeScript, etc.] |
| **Last Updated** | [DD Month YYYY] |

---

## 1. Naming Conventions
| Element | Convention | Example |
|---------|-----------|---------|
| Variables | snake_case / camelCase | `user_name` |
| Functions | snake_case / camelCase | `get_user()` |
| Classes | PascalCase | `UserService` |
| Constants | UPPER_SNAKE | `MAX_RETRIES` |
| Files | snake_case / kebab-case | `user_service.py` |

## 2. Formatting
- Indentation: [spaces/tabs, count]
- Max line length: [80/120]
- Import ordering: [stdlib, third-party, local]
- Linter: [tool and config file]
- Formatter: [tool and config file]

## 3. Error Handling
[Convention for try/catch, custom exceptions, error propagation.]

## 4. Logging
[Log levels, format, what to log, what NOT to log.]

## 5. Git Conventions
### Commit Messages
```
<type>(<scope>): <subject>

Types: feat, fix, docs, style, refactor, test, chore
```

### PR Requirements
- [ ] Description of changes
- [ ] Tests added/updated
- [ ] Linter passes
- [ ] At least 1 reviewer approval
