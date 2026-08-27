# Context Template

The output file `.context/hu-<code>.md` must follow this structure.

```markdown
# Context: <HU title>

## 1. Metadata

| Field | Value |
| --- | --- |
| Code | `<code>` |
| Actor | `<actor>` |
| Action | `<action>` |
| Goal | `<goal>` |
| Workspace | `<path>` |
| Generated | `<ISO-8601>` |

## 2. Acceptance Criteria

| ID | Functional text | Technical hint |
| --- | --- | --- |
| CA1 | ... | ... |

## 3. Business Rules

- BR1: ...

## 4. Services

### 4.1 Primary

| Service | Role | Injection point |
| --- | --- | --- |
| `service-name` | ... | `Class.method` |

### 4.2 Secondary

| Service | Role | Impact |
| --- | --- | --- |
| `service-name` | ... | ... |

### 4.3 Omitted

| Service | Reason |
| --- | --- |
| `service-name` | ... |

### 4.4 To Confirm

| Service | Reason |
| --- | --- |
| `service-name` | ... |

## 5. Current Technical Flow

1. Step 1...
2. Step 2...

## 6. Key Finding

<Most important discovery. Be specific.>

## 7. Expected Changes

| Service | Layer | File | Change |
| --- | --- | --- | --- |
| `service-name` | input | `*.java` | Add field X |

## 8. Affected Contracts

| Type | Location | Change |
| --- | --- | --- |
| REST | `POST /path` | Add field X |
| ASB | `queue-name` | Add field X to payload |

## 9. Migrations / Data Fixes

- Migration 1...

## 10. Gaps

| ID | Topic | Reason | Impact | Blocking |
| --- | --- | --- | --- | --- |
| G1 | ... | ... | ... | Yes / No |

## 11. Assumptions

- ASSUMED: ...

## 12. Risks

- RISK: ...
```
