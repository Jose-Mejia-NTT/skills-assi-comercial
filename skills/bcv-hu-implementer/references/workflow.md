# Workflow

## 0. Pre-conditions

Before running this skill, ensure:

- The DHU exists and has no unresolved blocking gaps.
- The context file `.context/hu-<code>.md` exists.
- All affected repositories have pre-built `graphify-out/graph.json`.
- The workspace context files exist (`service-map.md`, `gotchas.md`).

If the DHU has blocking gaps, **stop** and ask the user to resolve them first.

---

## 1. Read inputs

1. Read the DHU file.
2. Read the context file if provided.
3. Extract the technical implementation map from the DHU.

---

## 2. Determine affected repositories

From the DHU's technical map, list each affected microservice:

```text
bcv-bacc-party-lifecycle-management-service
bcv-bacc-channel-activity-service
...
```

Skip repositories marked as `to_confirm` or `omitted` unless the DHU explicitly assigns them work.

---

## 3. Discover skills available to GitHub Copilot Chat (mandatory)

Before writing the report, discover **all** skills that Copilot Chat can access, both in the workspace and in the user's environment. Use a glob so every `SKILL.md` is picked up:

```bash
find . -type f -name 'SKILL.md'
find "$HOME/skills" -type f -name 'SKILL.md' 2>/dev/null
find "$HOME/.config/github-copilot/skills" -type f -name 'SKILL.md' 2>/dev/null
```

Also read `.github/copilot-instructions.md` (workspace) and `~/.github/copilot-instructions.md` (user) and collect any skill names mentioned there.

Collect the `name:` field from each `SKILL.md` found. This is the **available skill set** (`*`), combining workspace + user. Reuse it for every row in the report.

---

## 4. Generate the implementation report (single file)

Write ONE file:

```
hu-technical-refinement/HU-<code>-implementation-report-<YYYYMMDDHHmm>.md
```

Follow [output-template.md](output-template.md). For each file change / task, add a `Skill (cómo codificar)` column with the recommended skill from the available set and a one-line convention from its `Dictates` column in [skill-references.md](skill-references.md). The referenced skill is the authority for HOW the code must be written. If none matches, write `not available in Copilot Chat`.

The report includes:

- Pre-implementation validation block.
- Summary metrics.
- Per-repository sections (files modified/created with `Skill` column, diff excerpts).
- Manual tasks remaining.
- Next steps.

This file is generated in both `dry-run` and `apply` modes. Do **not** create a separate draft file.

---

## 5. For each affected repository (only in `apply` mode)

### 5.1 Create feature branch

```bash
cd <repo>
git checkout -b feature/HU-<code>
```

If a branch with the same name already exists, append a timestamp or short suffix.

### 5.2 Read existing files

Read only the files listed in the technical map. Use `read offset/limit` to keep reads minimal.

Do not read files that are not listed in the map unless necessary to understand a dependency.

### 5.3 Generate changes

For each file in the map, generate the required change.

**Before generating each change**, read and apply the referenced skill (Camino B):

1. Read the referenced skill's `SKILL.md`.
2. Read the relevant `references/*.md` of that skill.
3. Apply its mandatory rules and conventions to the generated snippet.

The change types and their typical reference skills are:

| Change type | Action | Reference skill (hint) |
| --- | --- | --- |
| Add field to DTO/record | Append field with type and validation annotations. | `bcv-openapi-design` |
| Add field to entity | Append field with JPA annotations. | `bcv-spring-data-jpa-sql-server` |
| Add validation | Add method or annotation-based validation. | `bcv-java-spring-boot` |
| Update mapper | Add mapping for the new field. | `bcv-hexagonal-architecture` or `bcv-clean-architecture` |
| Update use case | Add business logic step. | `bcv-hexagonal-architecture` or `bcv-clean-architecture` |
| Update controller | Add endpoint or modify existing one. | `bcv-openapi-design` |
| Add repository | Create new repository interface if needed. | `bcv-spring-data-jpa-sql-server` |
| Add migration | Create Flyway/Liquibase script if needed. | `bcv-spring-data-jpa-sql-server` |
| Add tests | Create or update unit/integration tests. | `bcv-java-spring-boot` |

Generate snippets, not entire files, unless the file is new.

Apply the **code quality rules** to every generated snippet: **no comments in the generated code** (no `//`, no `/* */`, no `TODO`; remove existing comments on modified lines), no commented-out code, no unused imports, no magic numbers, no `System.out`/`printStackTrace`, and consistent naming — so SonarQube reports zero alerts.

### 5.4 Apply changes (only in `apply` mode)

In `dry-run` mode, skip this step and show the diff in the report.

In `apply` mode:

1. Write modifications to existing files.
2. Create new files.
3. Do not delete existing code unless explicitly required by the DHU.

### 5.5 Build, test and run (Maven)

Use the Maven wrapper (`./mvnw`) from the repository root. Multi-module projects target a single module with `-pl <module>`.

| Acción | Comando |
| --- | --- |
| Compilar | `./mvnw clean compile` |
| Tests unitarios | `./mvnw test` |
| Tests + integración + linter + empaquetar | `./mvnw verify` |
| Empaquetar sin tests | `./mvnw clean package -DskipTests` |
| Ejecutar la app (módulo `-app`) | `./mvnw -pl <nombre>-app spring-boot:run` |
| Ejecutar el jar empaquetado | `java -jar <nombre>-app/target/<nombre>-app-*.jar` |

If `verify` (linter/tests) fails, stop and report the error. Do not proceed to the next repo.

In `apply` mode, run `./mvnw verify` per repository and record the result in the report. Include the exact `spring-boot:run` / `java -jar` command for the `-app` module in the "Build / run commands" section of the report.

---

## 6. Respond to the user

Return a concise summary with:

- Path to the implementation report (single file).
- Mode used (`dry-run` or `apply`).
- Repositories affected.
- Branch names.
- Number of files modified/created.
- Linter and test results.
- Next steps for the developer (review, commit, push).
