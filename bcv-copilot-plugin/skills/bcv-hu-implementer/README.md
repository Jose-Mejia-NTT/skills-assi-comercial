# bcv-hu-implementer

Applies implementation changes derived from a refined technical HU (DHU) into the affected backend microservice repositories.

## When to use

- After `bcv-dhu-writer` has produced an approved DHU.
- When the DHU has no unresolved blocking gaps.
- When you want to bootstrap implementation from the DHU.

## When NOT to use

- If the DHU has unresolved blocking gaps.
- For frontend or UI implementation.
- As a replacement for human code review.

## Companion skills

- `bcv-hu-context-analyzer` — generates `.context/hu-<code>.md`.
- `bcv-dhu-writer` — generates `hu-technical-refinement/HU-...-refined-...md`.
- `bcv-hu-implementer` — applies implementation changes (this skill).

## Main files

- `SKILL.md` — entry point and overview.
- `references/workflow.md` — step-by-step implementation process.
- `references/limits.md` — read, query, and modification limits.
- `references/output-template.md` — implementation report format.
- `references/example.md` — complete walkthrough.

## Safety first

This skill works in `dry-run` mode by default. It only shows proposed diffs.
To apply changes, the user must explicitly request `apply` mode.
