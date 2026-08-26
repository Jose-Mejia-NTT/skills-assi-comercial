# Implementation task to skill mapping

This reference maps each `IMP-XXX` task type to the appropriate BCV specialized
implementation skill. The orchestrator uses this table to select the right skill
and to build a self-contained prompt for it.

## Default mapping (Java/Spring Boot stack)

| Task type | Specialized skill | Responsibility |
|---|---|---|
| `discovery` | `bcv-business-resolution` or `bcv-technical-impact-and-story` | Resolve ambiguities before implementation. |
| `contract` | `bcv-openapi-design` | Design or update OpenAPI specs, DTOs, event schemas. |
| `domain` (backend) | `bcv-java-spring-boot` | Implement controllers, use cases, domain services, handlers. |
| `domain` (architecture setup) | `bcv-clean-architecture` or `bcv-hexagonal-architecture` | Define project structure, layers, ports/adapters. |
| `persistence` (SQL Server) | `bcv-spring-data-jpa-sql-server` | Repositories, entities, migrations, queries. |
| `persistence` (Cosmos DB) | `bcv-cosmos-db` | Cosmos entities, containers, repositories. |
| `messaging` / `events` | `bcv-azure-service-bus` | Topics, subscriptions, publishers, consumers. |
| `observability` | `bcv-commons-observability` | Logs, metrics, tracing, health checks. |

## Stack override

If the user specifies a different stack, update the mapping:

- `nodejs` → replace `bcv-java-spring-boot` with the appropriate Node skill.
- `python` → replace with the appropriate Python skill.
- If no stack is specified, default to Java/Spring Boot.

## Prompt building rules

When building a prompt for a specialized skill, always include:

1. The original HU summary.
2. The `IMP-XXX` task description and step-by-step implementation.
3. The acceptance criteria from the detailed action plan.
4. The files affected and the expected output files.
5. A reference to `technical-impact-analysis.yaml` for context.
6. The instruction to not invent requirements outside the story.

## Manual tasks

If a task type does not match any available specialized skill, mark it as
`MANUAL` in the orchestration plan and provide guidance for the developer.
