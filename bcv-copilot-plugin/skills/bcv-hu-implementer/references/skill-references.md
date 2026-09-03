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

The `Dictates (how to code)` column tells the developer/Copilot Chat the concrete conventions that skill enforces, so the referenced skill is not just a label but the authority for HOW the code must be written.

| Task type                                                 | Recommended skill                 | When to use                                                                                                    | Dictates (how to code)                                                                                                                                          |
| --------------------------------------------------------- | --------------------------------- | -------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Define or refactor hexagonal packages (input/core/output) | `bcv-hexagonal-architecture`      | Creating controllers, use cases, ports, domains, repositories, publishers, subscribers in a hexagonal service. | Input solo adapta; core tiene la lógica; output implementa puertos. Mappers MapStruct singletons `X.MAPPER`. Command controllers con `@StandardApi` + RFC 9457. |
| Define or refactor clean architecture layers              | `bcv-clean-architecture`          | When the service follows clean architecture or you need to enforce dependency rules.                           | Dependencias apuntan al centro (entities/use cases); nada de infraestructura en el dominio.                                                                     |
| Create or modify Spring Boot components                   | `bcv-java-spring-boot`            | Beans, configuration, dependency injection, profiles, application properties, starters, validation.            | Beans `@Component`/`@Service`; validación con Jakarta annotations; perfiles para config por entorno.                                                            |
| Design or update REST/OpenAPI contracts                   | `bcv-openapi-design`              | Controllers, request/response records, error envelopes, OpenAPI specs.                                         | Records con `@Schema`; errores RFC 9457; códigos HTTP correctos; contrato documentado en OpenAPI.                                                               |
| Implement Azure Service Bus messaging                     | `bcv-azure-service-bus`           | Topics, subscriptions, publishers, subscribers, message handlers, ASB configuration.                           | Handlers idempotentes; payloads con `X-Request-Id`; reintentos y DLQ configurados.                                                                              |
| Implement SQL Server persistence with Spring Data JPA     | `bcv-spring-data-jpa-sql-server`  | Entities, repositories, queries, transactions, Flyway/Liquibase migrations.                                    | Entidades `@Entity` + `@Column`; repositorios Spring Data; migraciones Flyway versionadas `V{ts}__`.                                                            |
| Implement Cosmos DB persistence                           | `bcv-cosmos-db`                   | Cosmos containers, documents, repositories, queries, SDK usage.                                                | Partición por clave definida; POJOs con `@PartitionKey`; no queries entre particiones.                                                                          |
| Add observability (logs, metrics, tracing)                | `bcv-commons-observability`       | Micrometer, OpenTelemetry, structured logging, correlation IDs.                                                | Logs estructurados JSON; metrics con Micrometer; propaga `traceId`/`correlationId`.                                                                             |
| Resolve business rules or decision tables                 | `bcv-business-resolution`         | Complex business rules, conditions, decision services.                                                         | Reglas en clase de dominio o servicio de resolución; sin lógica de negocio en controladores.                                                                    |
| Estimate impact and split stories                         | `bcv-technical-impact-and-story`  | When the draft reveals that the HU should be split or re-scoped.                                               | Divide HUs por responsabilidad; máximo 8 CAs por HU.                                                                                                            |
| Orchestrate multi-service implementation                  | `bcv-implementation-orchestrator` | When changes span several services and need coordination.                                                      | Una rama `feature/HU-<code>` por repo; misma convención de commits; PRs coordinados.                                                                            |

## How the referenced skill is used (Camino B)

The `Skill` column is **not just a label**. For each task, before generating the code, the implementer must:

1. Read the referenced skill's `SKILL.md`.
2. Read the relevant `references/*.md` of that skill (e.g., `hexagonal-rules.md`, `controller-api-conventions.md`, `auditing.md`).
3. Apply its mandatory rules and conventions to the generated snippet.

The skill's location is the one discovered in the discovery step (workspace or user-level). The `Dictates` column above is only a **one-line hint**; the actual authority is the skill's full `SKILL.md` + `references/`.

If the referenced skill's files cannot be read (not present or not accessible), fall back to the `Dictates` hint and mark the task with `not available in Copilot Chat`.

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
