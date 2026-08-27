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

For each affected repository, the skill creates a feature branch and applies:

- Source code changes in existing files (snippets/diffs).
- New classes, mappers, repositories, or entities when needed.
- Unit and integration test snippets.
- Database migrations when needed.
- Updated DTOs, records, or event payloads.

The skill returns:

- List of affected repositories.
- Feature branch name.
- Files modified / created per repository.
- Linter and test results.
- A `git diff` summary for human review.

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

See [references/output-template.md](references/output-template.md) for the implementation report format.

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
