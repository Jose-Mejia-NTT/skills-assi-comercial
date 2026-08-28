---
name: bcv-hu-implementer
description: |
  Consume a refined DHU and its technical context, then generate and apply implementation
  changes in the affected backend microservice repositories. Creates feature branches,
  modifies source files, generates tests and migrations, runs linter and tests, and
  presents the diff for human review. Never commits or pushes automatically.
  Triggers: "implement this HU", "generate code for HU", "hu-implementer", "/hu-implement", "apply DHU".
  Do NOT perform business analysis or write DHUs. Use bcv-hu-context-analyzer and bcv-dhu-writer first.
argument-hint: "Path to hu-technical-refinement/HU-...-refined-...md or .context/hu-<code>.md"
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "implementation"
  frameworks: ["Graphify", "BMAD", "Spec-Driven Development"]
---

# bcv-hu-implementer

## Objective

Consume a refined technical HU (DHU) and its companion context file, then generate and apply implementation changes in the affected backend microservice repositories.

This skill is the third and final step of the HU → DHU → Code pipeline:

1. `bcv-hu-context-analyzer` → generates technical context.
2. `bcv-dhu-writer` → writes the refined DHU.
3. `bcv-hu-implementer` → applies implementation changes in feature branches.

## Scope

- **Backend microservices only.** This skill does not generate frontend, UI, or presentation code.
- Works only from the DHU and context file. It does not re-analyze the HU.
- Modifies files inside the affected repositories, not in a separate folder.
- Never commits or pushes to `main` automatically.

## Mandatory inputs

1. Path to the refined DHU:
   ```
   hu-technical-refinement/HU-{identifier}-refined-{YYYYMMDDHHmm}.md
   ```
2. Path to the context file (optional if DHU includes all required context):
   ```
   .context/hu-<code>.md
   ```
3. Workspace path with the affected repositories. If omitted, use the current workspace.

## Output

The skill generates a **single** Markdown file: the implementation report. It does **not** create a separate draft file.

```
hu-technical-refinement/HU-<code>-implementation-report-<YYYYMMDDHHmm>.md
```

This report contains:

- Pre-implementation validation block.
- Summary metrics.
- Per-repository sections with files modified/created, each row annotated with the recommended `Skill`.
- Manual tasks remaining.
- Next steps.

The report is generated in both `dry-run` and `apply` modes.

#### Applied changes (only in `apply` mode)

For each affected repository, the skill creates a feature branch and applies:

- Source code changes in existing files (snippets/diffs).
- New classes, mappers, repositories, or entities when needed.
- Unit and integration test snippets.
- Database migrations when needed.
- Updated DTOs, records, or event payloads.

#### Skill discovery (mandatory, before writing the report)

Discover **all** skills available to GitHub Copilot Chat, both in the workspace and in the user's personal environment:

1. **Workspace:** `find . -type f -name 'SKILL.md'` (or `<workspace>/**/SKILL.md`).
2. **User level:** scan the user's personal skill folders, e.g. `~/skills`, `~/.config/github-copilot/skills`, or any folder the user declares as their skills root.
3. Read `.github/copilot-instructions.md` (workspace) and `~/.github/copilot-instructions.md` (user) and collect skill names mentioned there.
4. Collect the `name:` field from each `SKILL.md` found. This is the **available skill set** (`*`), combining workspace + user.
5. For each file change / task, pick the skill from the available set that best matches the work (use [skill-references.md](references/skill-references.md) as a hint for the mapping). The referenced skill is the **authority for HOW the code must be written** for that task — include a one-line convention from its `Dictates` column in the report.
6. If no available skill matches a task, mark it as `not available in Copilot Chat` and keep the task description so the user can implement it manually.

## Execution modes

| Mode      | Condition                                 | Output                                            |
| --------- | ----------------------------------------- | ------------------------------------------------- |
| `dry-run` | Default if no flag is given               | Proposed changes shown as diff, no files modified |
| `apply`   | User explicitly requests to apply changes | Changes applied in feature branches               |

Default mode is `dry-run` to prevent accidental modifications.

## Safety rules

1. **Always create a feature branch.** Never modify `main` or an existing protected branch.
2. **Never commit or push automatically.** The user must review and commit manually.
3. **Only touch files listed in the DHU's technical map.** Do not invent new files.
4. **Run linter and tests** after applying changes in `apply` mode.
5. **Stop on test or linter failure** and report the error.
6. **Preserve existing code style.** Match indentation, naming, and conventions of the repo.

## Workflow

See [references/workflow.md](references/workflow.md) for the step-by-step process.

## Limits

See [references/limits.md](references/limits.md) for read, query, and modification limits.

## Output template

- [references/output-template.md](references/output-template.md) — format of the single implementation report (with skill column).

## Language handling and output policy

See [references/language-policy.md](references/language-policy.md).

## Example

See [references/example.md](references/example.md) for a complete walkthrough.

## When to use

- After `bcv-dhu-writer` has produced an approved DHU.
- When the DHU has no unresolved blocking gaps.
- When the team wants to bootstrap implementation from the DHU.

## When NOT to use

- If the DHU has blocking gaps unresolved.
- If the DHU fails quality validation.
- For frontend or UI implementation.
- As a replacement for human code review.

## Pre-implementation validation

Before applying any change, the skill must validate the DHU:

| Check               | Rule                                                          |
| ------------------- | ------------------------------------------------------------- |
| Blocking gaps       | No unresolved blocking gaps allowed.                          |
| DHU state           | State must not be `EN ELABORACIÓN` if gaps remain.            |
| Acceptance criteria | At least 3 technical acceptance criteria present.             |
| Technical map       | `Mapa técnico de implementación` must list files and changes. |
| DoR                 | All DoR items must be checked or justified.                   |

If validation fails, the skill stops and returns a report with the issues. No files are modified, even in `apply` mode.

## Language handling and output policy

See [references/language-policy.md](references/language-policy.md).

## References

- [workflow.md](references/workflow.md)
- [limits.md](references/limits.md)
- [output-template.md](references/output-template.md)
- [language-policy.md](references/language-policy.md)
- [example.md](references/example.md)
