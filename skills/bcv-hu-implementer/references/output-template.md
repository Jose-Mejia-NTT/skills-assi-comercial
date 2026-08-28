> This template defines the exact public output format for the implementation report.

````markdown
# Implementation Report — HU-<code>

> **Mode:** {dry-run | apply}
> **DHU source:** `hu-technical-refinement/HU-<code>-refined-...md`
> **Context source:** `.context/hu-<code>.md`
> **Implementation draft:** `.context/implementation-draft-HU-<code>.md`
> **Generated:** {YYYY-MM-DD HH:mm}

---

## Summary

| Metric | Value |
| --- | --- |
| Repositories affected | {N} |
| Feature branches created | {N} |
| Files modified | {N} |
| Files created | {N} |
| Tests created/updated | {N} |
| Migrations created | {N} |
| Linter status | {passed / failed} |
| Test status | {passed / failed} |

---

## Implementation draft summary

The draft file `.context/implementation-draft-HU-<code>.md` contains the detailed task plan with recommended skills.

### Tasks by service

| # | Service | Layer | Task | Skill | Blocked by |
|---|---|---|---|---|---|
| 1 | `{service}` | `{layer}` | {task description} | `{skill-name}` | {gap or N/A} |
| 2 | `{service}` | `{layer}` | {task description} | `not available locally` | {gap or N/A} |

> Only skills installed on the user's machine are referenced by name. If a recommended skill is not installed, the row shows `not available locally`.

### Skill legend

| Skill | Used for | Available locally |
|---|---|---|
| `bcv-hexagonal-architecture` | Hexagonal package structure, ports, adapters, mappers | {yes / no} |
| `bcv-clean-architecture` | Clean architecture layers and dependency rules | {yes / no} |
| `bcv-java-spring-boot` | Spring components, beans, validation, tests | {yes / no} |
| `bcv-openapi-design` | REST controllers, request/response records, OpenAPI | {yes / no} |
| `bcv-azure-service-bus` | ASB publishers, subscribers, message handlers | {yes / no} |
| `bcv-spring-data-jpa-sql-server` | JPA entities, repositories, migrations | {yes / no} |
| `bcv-cosmos-db` | Cosmos DB documents, repositories, queries | {yes / no} |
| `bcv-commons-observability` | Logs, metrics, tracing | {yes / no} |
| `bcv-business-resolution` | Business rules and decision logic | {yes / no} |
| `bcv-technical-impact-and-story` | Story splitting and impact analysis | {yes / no} |
| `bcv-implementation-orchestrator` | Multi-service coordination | {yes / no} |

---

## Repositories

### {service-name}

**Branch:** `feature/HU-<code>`

**Status:** {ready / failed / partial}

**Files modified:**

| File | Change |
| --- | --- |
| `src/main/java/.../{File}.java` | {brief description} |
| `src/main/java/.../{Mapper}.java` | {brief description} |

**Files created:**

| File | Purpose |
| --- | --- |
| `src/main/java/.../{NewClass}.java` | {brief description} |
| `src/test/java/.../{NewTest}.java` | {brief description} |
| `src/main/resources/db/migration/V...__.sql` | {brief description} |

**Validation:**

```text
Linter: {passed / failed}
Tests: {N} passed, {N} failed
````

**Diff summary:**

```diff
{short diff excerpt}
```

---

## Manual tasks remaining

- [ ] {task 1}
- [ ] {task 2}

---

## Next steps

1. Review the implementation draft `.context/implementation-draft-HU-<code>.md`.
2. For each task, invoke the recommended skill if additional guidance is needed.
3. Review the generated diff in each repository.
4. Run tests locally if not already run.
5. Commit and push the feature branches manually.
6. Create pull requests for human review.

```

```
