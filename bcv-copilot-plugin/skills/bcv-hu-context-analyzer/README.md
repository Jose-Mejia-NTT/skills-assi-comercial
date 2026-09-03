# bcv-hu-context-analyzer

Investigates a BCV business user story across one or more microservice repositories using graphify CLI, producing a structured technical context file with minimal token usage.

## When to use

- Before writing a technical specification (DHU).
- When a HU spans multiple repositories.
- When you need to identify the injection point and affected services.

## When NOT to use

- For business/functional resolution (use `bcv-business-resolution`).
- For final DHU writing (use `bcv-dhu-writer`).
- For implementation or test generation.

## Companion skill

- `bcv-dhu-writer` — consumes `.context/hu-<code>.md` and writes `docs/hu-tecnicas/<code>.md`.

## Required context

The skill needs very little written context:

1. **Workspace-level:** `docs/.agent-context/service-map.md` (role, stack, god nodes per service, < 100 lines).
2. **Service-level (optional):** `<repo>/docs/.agent-context/gotchas.md` (non-obvious constraints, < 10 lines).

No per-service baseline cache or full README reads are required.

## Maintaining graphify graphs

The skill assumes each repository has a pre-built `graphify-out/graph.json`. Keep them fresh with this rule of thumb:

| Update the graph when... | Which repo? |
|---|---|
| New controller, subscriber, publisher, or use case | The service where it was added |
| New database entity or table | The service that owns the entity |
| New Feign client or external integration | The consuming service |
| Rename/removal of a god node | The affected service |
| Significant package refactor | The affected service |
| Merge to `main` with structural changes | All affected services (prefer CI) |

**No rebuild needed** for bugfixes inside existing methods, new fields in existing DTOs/entities, new tests, or documentation-only changes.

Use `graphify check-update <repo-path>` to verify if a graph is stale.

For full details, see `references/optimizations.md`.

## Main files

- `SKILL.md` — entry point and overview.
- `references/workflow.md` — step-by-step process.
- `references/optimizations.md` — token-saving optimizations and graph maintenance policy.
- `references/output-template.md` — Markdown output structure.
- `references/language-policy.md` — user-facing vs internal language rules.
- `references/gap-handling.md` — how to handle unknowns.
- `references/example.md` — complete example.
