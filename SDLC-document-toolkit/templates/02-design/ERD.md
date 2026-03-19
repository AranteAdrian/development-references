# Entity Relationship Diagram (ERD)

| Field | Value |
|-------|-------|
| **System** | [System Name] |
| **Author** | [Name] |
| **Date** | [DD Month YYYY] |

---

## 1. ER Diagram
```mermaid
erDiagram
    ENTITY_A ||--o{ ENTITY_B : "relationship"
    ENTITY_A {
        uuid id PK
        string name
        timestamp created_at
    }
    ENTITY_B {
        uuid id PK
        uuid entity_a_id FK
        string value
    }
```

## 2. Entity Descriptions
| Entity | Description | Key Attributes |
|--------|-------------|---------------|
| | | |

## 3. Relationship Summary
| From | To | Type | Description |
|------|----|------|-------------|
| | | 1:N / M:N / 1:1 | |

## 4. Indexes
| Table | Index Name | Columns | Type | Rationale |
|-------|-----------|---------|------|-----------|
| | | | B-tree/Hash | |
