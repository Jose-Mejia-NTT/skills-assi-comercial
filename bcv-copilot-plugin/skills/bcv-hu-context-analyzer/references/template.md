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

| ID | Type | Blocking | Description | Suggested answer | Affected DHU section |
| --- | --- | --- | --- | --- | --- |
| GAP-01 | business | yes | Catalog source not defined. | Local table `registry_office` in PLM. | Endpoint, persistence |
| GAP-02 | technical | yes | Field location unclear. | `BusinessAccountRecord.registryOffice`. | Request schema, entity |
| GAP-03 | implementation | no | Exact JSON key name. | `registryOffice` in English. | DTO, contract |

### Gap types

- `business` — requires product/business clarification.
- `technical` — requires architecture or service owner clarification.
- `implementation` — can be decided by the development team.

### Blocking

- `yes` — prevents completing the DHU; must be resolved before implementation.
- `no` — can be decided later or assumed with a default.

## 11. Assumptions

- ASSUMED: ...

## 12. Risks

- RISK: ...
```
