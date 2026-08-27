> This template defines the exact public output format for the implementation report.

```markdown
# Implementation Report — HU-<code>

> **Mode:** {dry-run | apply}
> **DHU source:** `hu-technical-refinement/HU-<code>-refined-...md`
> **Context source:** `.context/hu-<code>.md`
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
```

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

1. Review the generated diff in each repository.
2. Run tests locally if not already run.
3. Commit and push the feature branches manually.
4. Create pull requests for human review.
```
