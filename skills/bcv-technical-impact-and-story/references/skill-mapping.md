# Skill mapping for implementation tasks

This reference maps each task type in the technical task plan to a **skill category**.
Each user must map these categories to the actual skills available in their assistant ecosystem.

## How to use this mapping

1. When generating the technical task plan, assign a `Recommended skill category` to each task.
2. If the user provides a list of available skills, replace the category with the matching skill name.
3. If no skill list is available, keep the category and add a note like:
   `Replace with the user's skill for [category]`.

## Task type → skill category

| Task type | Skill category | Typical responsibility |
|---|---|---|
| `discovery` | `hu-analysis` | Resolve ambiguities, conflicts or missing information before implementation. |
| `contract` | `contract-design` | Design or modify REST/OpenAPI contracts, endpoints, request/response schemas. |
| `domain` (backend) | `backend-dev` | Implement business logic, use cases, services, controllers, subscribers. |
| `domain` (frontend) | `frontend-dev` | Implement web components, pages or user interfaces. |
| `domain` (mobile) | `mobile-dev` | Implement mobile-first experiences or mobile apps. |
| `persistence` | `database-dev` | Database design, migrations, repositories, queries. |
| `messaging` / `events` | `messaging-dev` | Event contracts, publishers/consumers, messaging integrations. |
| `observability` | `observability-dev` | Tracing, metrics, logging, health checks, alerts. |
| `testing` | `testing-dev` | Unit, integration and e2e test design and implementation. |
| `security` | `security-review` | Security review, authentication/authorization, compliance checks. |
| `documentation` | `docs-dev` | Write specs, READMEs, API documentation or runbooks. |
| `exploration` | `code-exploration` | Explore existing code, architecture and file relationships. |

## User skill mapping example

If the user reports these available skills:

```yaml
available_skills:
  hu-analysis: bcv-business-resolution
  contract-design: api-design
  backend-dev: bcv-java-spring-boot
  database-dev: bcv-spring-data-jpa-sql-server
  messaging-dev: bcv-azure-service-bus
  observability-dev: bcv-commons-observability
  testing-dev: bcv-java-spring-boot
```

Then a `domain` task in a Java/Spring service would show:

```markdown
- **Recommended skill:** `bcv-java-spring-boot`
```

## Default behavior

If no mapping is provided, use the category name as a placeholder and keep the
recommendation generic. Do not invent skill names that the user has not confirmed.
