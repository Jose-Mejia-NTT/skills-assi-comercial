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

## 3. For each affected repository

### 3.1 Create feature branch

```bash
cd <repo>
git checkout -b feature/HU-<code>
```

If a branch with the same name already exists, append a timestamp or short suffix.

### 3.2 Read existing files

Read only the files listed in the technical map. Use `read offset/limit` to keep reads minimal.

Do not read files that are not listed in the map unless necessary to understand a dependency.

### 3.3 Generate changes

For each file in the map, generate the required change:

| Change type | Action |
| --- | --- |
| Add field to DTO/record | Append field with type and validation annotations. |
| Add field to entity | Append field with JPA annotations. |
| Add validation | Add method or annotation-based validation. |
| Update mapper | Add mapping for the new field. |
| Update use case | Add business logic step. |
| Update controller | Add endpoint or modify existing one. |
| Add repository | Create new repository interface if needed. |
| Add migration | Create Flyway/Liquibase script if needed. |
| Add tests | Create or update unit/integration tests. |

Generate snippets, not entire files, unless the file is new.

### 3.4 Apply changes (only in `apply` mode)

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

## 4. Generate implementation report

Write a report following [output-template.md](output-template.md).

The report must include:

- Affected repositories.
- Feature branch names.
- Files modified and created per repository.
- Linter/test results.
- Remaining manual tasks.
- Full `git diff` summary.

---

## 5. Respond to the user

Return a concise summary with:

- Mode used (`dry-run` or `apply`).
- Repositories affected.
- Branch names.
- Number of files modified/created.
- Linter and test results.
- Next steps for the developer (review, commit, push).
