# API Specification

| Field | Value |
|-------|-------|
| **Service** | [Service Name] |
| **Base URL** | `https://api.example.com/v1` |
| **Auth** | Bearer token / API key / OAuth2 |
| **Version** | v1 |

---

## 1. Authentication
[How to obtain and use credentials.]

## 2. Common Headers
| Header | Required | Value |
|--------|----------|-------|
| `Authorization` | Yes | `Bearer <token>` |
| `Content-Type` | Yes | `application/json` |

## 3. Endpoints

### 3.1 [Resource Name]

#### GET /resource
**Description:** [What it does]

**Parameters:**
| Param | In | Type | Required | Description |
|-------|----|------|----------|-------------|
| | query/path | string | Yes/No | |

**Response 200:**
```json
{
  "id": "string",
  "name": "string"
}
```

**Error Responses:**
| Status | Description |
|--------|-------------|
| 400 | Bad request |
| 401 | Unauthorized |
| 404 | Not found |

## 4. Rate Limiting
| Tier | Limit | Window |
|------|-------|--------|
| Standard | [X] req | per minute |

## 5. Pagination
[Cursor-based / offset-based, page size defaults.]

## 6. Error Format
```json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "Description of the error"
  }
}
```
