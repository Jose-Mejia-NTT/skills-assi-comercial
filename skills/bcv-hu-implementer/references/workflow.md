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

## 3. Generate implementation draft

Create the draft file before modifying any repository:

```
.context/implementation-draft-HU-<code>.md
```

### 3.1 Discover skills available to GitHub Copilot Chat (mandatory)

Before assigning skills, discover **all** skills that Copilot Chat can access, both in the workspace and in the user's environment. Use a glob so every `SKILL.md` is picked up:

```bash
# Workspace skills (recursive)
find . -type f -name 'SKILL.md'

# User-level skills (personal folders)
find "$HOME/skills" -type f -name 'SKILL.md' 2>/dev/null
find "$HOME/.config/github-copilot/skills" -type f -name 'SKILL.md' 2>/dev/null
# Or any folder the user declares as their personal skills root
```

Also read `.github/copilot-instructions.md` (workspace) and `~/.github/copilot-instructions.md` (user) and collect any skill names mentioned there.

Collect the `name:` field from each `SKILL.md` found. This is the **available skill set** (`*`), combining workspace + user. Every skill discovered this way is eligible to be referenced in the draft.

### 3.2 Build the draft

The draft must include:

- All tasks from the DHU technical map, grouped by service and layer.
- For each task, a `Skill` column with the recommended skill from [skill-references.md](skill-references.md) **only if it is in the local skill set**.
- If the recommended skill is not installed locally, write `not available locally` and keep the task description.
- Files to modify/create per repository.
- Blockers and manual tasks.

Example task row:

```markdown
| # | Service | Layer | Task | Skill | Blocked by |
|---|---|---|---|---|---|
| 1 | plm | input | Add `registryOffice` field to request record | `bcv-openapi-design` | N/A |
| 2 | plm | core | Validate selected office against catalog | `bcv-business-resolution` | GAP-01 |
| 3 | plm | output | Persist `registryOffice` in `BusinessAccountRecord` | `bcv-spring-data-jpa-sql-server` | N/A |
| 4 | cas | subscriber | Forward `registryOffice` in SPL event payload | not available locally | N/A |
```

If the draft reveals that the HU should be split or re-scoped, annotate it with `bcv-technical-impact-and-story` **only if that skill is available locally**.

---

## 4. For each affected repository

### 4.1 Create feature branch

```bash
cd <repo>
git checkout -b feature/HU-<code>
```

If a branch with the same name already exists, append a timestamp or short suffix.

### 4.2 Read existing files

Read only the files listed in the technical map. Use `read offset/limit` to keep reads minimal.

Do not read files that are not listed in the map unless necessary to understand a dependency.

### 4.3 Generate changes

For each file in the map, generate the required change:

| Change type             | Action                                             | Reference skill                                          |
| ----------------------- | -------------------------------------------------- | -------------------------------------------------------- |
| Add field to DTO/record | Append field with type and validation annotations. | `bcv-openapi-design`                                     |
| Add field to entity     | Append field with JPA annotations.                 | `bcv-spring-data-jpa-sql-server`                         |
| Add validation          | Add method or annotation-based validation.         | `bcv-java-spring-boot`                                   |
| Update mapper           | Add mapping for the new field.                     | `bcv-hexagonal-architecture` or `bcv-clean-architecture` |
| Update use case         | Add business logic step.                           | `bcv-hexagonal-architecture` or `bcv-clean-architecture` |
| Update controller       | Add endpoint or modify existing one.               | `bcv-openapi-design`                                     |
| Add repository          | Create new repository interface if needed.         | `bcv-spring-data-jpa-sql-server`                         |
| Add migration           | Create Flyway/Liquibase script if needed.          | `bcv-spring-data-jpa-sql-server`                         |
| Add tests               | Create or update unit/integration tests.           | `bcv-java-spring-boot`                                   |

Generate snippets, not entire files, unless the file is new.

### 4.4 Apply changes (only in `apply` mode)

In `dry-run` mode, skip this step and show the diff.

In `apply` mode:

1. Write modifications to existing files.
2. Create new files.
3. Do not delete existing code unless explicitly required by the DHU.

### 3.5 Run linter and tests

```bash
./mvnw verify
```

Or the equivalent command for the repo's build tool.

If linter or tests fail, stop and report the error. Do not proceed to the next repo.

---

## 5. Generate implementation report

Write a report following [output-template.md](output-template.md).

The report must include:

- Path to the implementation draft.
- Affected repositories.
- Feature branch names.
- Files modified and created per repository.
- Linter/test results.
- Remaining manual tasks.
- Full `git diff` summary.

---

## 6. Respond to the user

Return a concise summary with:

- Path to the implementation draft.
- Mode used (`dry-run` or `apply`).
- Repositories affected.
- Branch names.
- Number of files modified/created.
- Linter and test results.
- Next steps for the developer (review, commit, push).
