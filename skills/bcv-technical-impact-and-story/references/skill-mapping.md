# Skill mapping for implementation tasks (Copilot)

This reference maps each task type in the technical task plan to a **skill category**.
In Copilot, the user must provide their available skills manually; the skill then
maps categories to those names.

## How Copilot provides skills

GitHub Copilot does not expose a standard skills directory or automatic skill list.
The user must provide their available skills in the chat prompt.

### Preferred input format

The user can write a message like:

```text
Mis skills disponibles en Copilot son:
- api-design
- bcv-java-spring-boot
- bcv-spring-data-jpa-sql-server
- bcv-azure-service-bus
- test-writer
```

Or as structured YAML:

```yaml
available_skills:
  contract-design: api-design
  backend-dev: bcv-java-spring-boot
  database-dev: bcv-spring-data-jpa-sql-server
  messaging-dev: bcv-azure-service-bus
  testing-dev: test-writer
```

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

## Reading skills from a path

If the user wants the skill to read the skill list from a path, they can provide it explicitly:

```text
Mis skills están en ./skills/
```

Supported path types in Copilot:

1. **Workspace-relative path** (recommended)
   - Example: `./skills/`, `.github/copilot-skills/`, `docs/skills/`
   - The skill can list files in that directory if it is inside the open workspace.
   - Skill names are inferred from file or folder names.

2. **Absolute local path** (limited)
   - Example: `C:\Users\me\skills\` or `/Users/me/skills/`
   - Copilot may not have access outside the workspace.
   - Use only if the assistant explicitly allows file-system access.

3. **GitHub repository URL**
   - Example: `https://github.com/my-org/copilot-skills`
   - The skill cannot browse the web unless the assistant has a web tool.
   - The user should paste the list instead.

### Behavior when a path is provided

1. List files or folders in the provided path.
2. Extract skill names from filenames (e.g. `api-design.md` → `api-design`).
3. Map those names to task categories using the table above.
4. If a category has no matching skill, fall back to the category placeholder.

If the path cannot be read, ask the user to paste the list manually.

## When the user does not provide skills or a path

If no skill list or path is provided, output **skill categories** and add a note like:

```markdown
- **Recommended skill category:** `backend-dev`
  - Replace with your Copilot skill for backend implementation.
```

Do not invent skill names that the user has not confirmed.
