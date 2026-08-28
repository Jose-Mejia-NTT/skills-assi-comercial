# Skill References for Implementation Tasks

This catalog maps common implementation task types to the corresponding skill in this repository. The `bcv-hu-implementer` uses it to annotate the implementation draft with the recommended skill for each task.

## Important: use only skills available to GitHub Copilot Chat

These BCV skills are consumed exclusively through **GitHub Copilot Chat** in the workspace. The draft must reference **only skills that GitHub Copilot Chat can access** in this workspace.

Before assigning a skill to a task, the implementer must discover which skills are available to Copilot Chat, both at workspace level and at user level:

1. **Workspace skills:** `find . -type f -name 'SKILL.md'` (or `<workspace>/**/SKILL.md`).
2. **User-level skills:** scan the user's personal skill folders, for example:
   - `~/skills/**/SKILL.md`
   - `~/.config/github-copilot/skills/**/SKILL.md`
   - Any folder the user declares as their personal skills root.
3. Read `.github/copilot-instructions.md` (workspace) and `~/.github/copilot-instructions.md` (user) and collect any skill names mentioned there.
4. Collect the `name:` field from each `SKILL.md` found. This is the **available skill set** (`*`), combining workspace + user.

If a recommended skill from the catalog below is **not** in the available set, the draft must:

1. Mark the skill as **not available in Copilot Chat**.
2. Keep the task description and files.
3. Optionally add a short note with the skill's purpose so the user can implement it manually.

## How to use

For each task in the implementation draft, pick the skill whose scope best matches the work **and that is available locally**. If a task spans multiple concerns, list the primary skill first and secondary skills in parentheses.

## Task → Skill mapping

| Task type                                                 | Recommended skill                 | When to use                                                                                                    |
| --------------------------------------------------------- | --------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Define or refactor hexagonal packages (input/core/output) | `bcv-hexagonal-architecture`      | Creating controllers, use cases, ports, domains, repositories, publishers, subscribers in a hexagonal service. |
| Define or refactor clean architecture layers              | `bcv-clean-architecture`          | When the service follows clean architecture or you need to enforce dependency rules.                           |
| Create or modify Spring Boot components                   | `bcv-java-spring-boot`            | Beans, configuration, dependency injection, profiles, application properties, starters, validation.            |
| Design or update REST/OpenAPI contracts                   | `bcv-openapi-design`              | Controllers, request/response records, error envelopes, OpenAPI specs.                                         |
| Implement Azure Service Bus messaging                     | `bcv-azure-service-bus`           | Topics, subscriptions, publishers, subscribers, message handlers, ASB configuration.                           |
| Implement SQL Server persistence with Spring Data JPA     | `bcv-spring-data-jpa-sql-server`  | Entities, repositories, queries, transactions, Flyway/Liquibase migrations.                                    |
| Implement Cosmos DB persistence                           | `bcv-cosmos-db`                   | Cosmos containers, documents, repositories, queries, SDK usage.                                                |
| Add observability (logs, metrics, tracing)                | `bcv-commons-observability`       | Micrometer, OpenTelemetry, structured logging, correlation IDs.                                                |
| Resolve business rules or decision tables                 | `bcv-business-resolution`         | Complex business rules, conditions, decision services.                                                         |
| Estimate impact and split stories                         | `bcv-technical-impact-and-story`  | When the draft reveals that the HU should be split or re-scoped.                                               |
| Orchestrate multi-service implementation                  | `bcv-implementation-orchestrator` | When changes span several services and need coordination.                                                      |

## Annotation format

In the implementation draft, each task row must include a `Skill` column:

```markdown
| # | Service | Layer | Task | Skill | Blocked by |
|---|---|---|---|---|---|
| 1 | plm | input | Add `registryOffice` to request record | `bcv-openapi-design` | N/A |
| 2 | plm | core | Validate office against catalog | `bcv-business-resolution` | GAP-01 |
| 3 | plm | output | Persist selection in `BusinessAccountRecord` | `bcv-spring-data-jpa-sql-server` | N/A |
| 4 | cas | subscriber | Include `registryOffice` in SPL payload | `bcv-azure-service-bus` | N/A |
```

## Notes

- These skills are guidance for the developer, not automatic execution steps.
- If a task does not match any skill, use `N/A` and add a note in the `Remarks` column.
- Keep this catalog updated when new skills are added to the repository.
