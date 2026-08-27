---
name: bcv-hu-context-analyzer
description: |
  Investigate a BCV business user story (HU/HAB) across one or more microservice repositories
  using graphify CLI, and produce a low-cost technical context file in Markdown.
  Triggers: "analyze this HU", "generate technical context", "hu-context", "context analyzer",
  "investigate HU", "pre-dhu analysis", "/hu-analyze".
  Do NOT generate implementation code, tests, migrations, or the final DHU.
  For final DHU generation, use bcv-dhu-writer (companion skill).
argument-hint: "HU/HAB text + workspace path or list of repositories"
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "technical-analysis"
  frameworks: ["Graphify", "BMAD", "Spec-Driven Development"]
---

# bcv-hu-context-analyzer

## Objective

Produce a structured technical context file (`.context/hu-<code>.md`) from a business user story and one or more BCV microservice repositories.

This skill is the first half of the HU → DHU pipeline:

1. `bcv-hu-context-analyzer` → generates technical context.
2. `bcv-dhu-writer` → consumes the context and writes the final DHU.

The skill minimizes token consumption by using **graphify CLI** instead of LLM-based repository radiography.

## Execution modes

| Mode          | Condition                                   | Output                                            |
| ------------- | ------------------------------------------- | ------------------------------------------------- |
| `single-repo` | One repository is provided or detected      | `.context/hu-<code>.md` for that repo             |
| `multi-repo`  | A workspace with multiple repos is provided | `.context/hu-<code>.md` consolidated across repos |

State the selected mode in the response.

## Mandatory inputs

1. Original HU/HAB text (title, actor, action, goal, acceptance criteria, business rules).
2. Repository path or workspace path. If omitted, use the current workspace.

## Output

A Markdown file:

```
.context/hu-<code>.md
```

where `<code>` is a kebab-case derivation of the HU title.

## Language handling and output policy

The skill must clearly separate **internal processing language** from **user-facing output language**.

### Language detection

1. At the start of every interaction, detect the language of the user's input.
2. The **user-facing responses** must always be written in the user's language.
3. If the user's language cannot be confidently detected, default to English.

### Output language (user-facing)

All content exposed to the user must be written in the user's language, including:

- Explanations and confirmations.
- Functional specifications.
- Acceptance criteria.
- Error messages visible to consumers.
- Documentation intended for human readers.
- User-facing validation or confirmation messages.

### Internal processing language

- For quality, consistency and technical accuracy, the skill may internally reason, plan, structure and code content in English.
- Internal reasoning language must never leak into the user-facing output.

### Technical artifacts language

To follow industry standards and best practices:

- **Source code**, **class names**, **method names**, **variable names** and **package names** must be written in English.
- **OpenAPI fields**, **JSON keys**, and **HTTP-level constructs** must be written in English.
- **Git commit messages** must be written in English unless the user explicitly requests otherwise.

### Important clarification

The skill template and internal logic may be defined in English, but **all responses visible to the user must respect the detected user language**.

## Workflow

See [references/workflow.md](references/workflow.md) for the step-by-step process.

## Token budget discipline

See [references/limits.md](references/limits.md) for strict query and read limits.

## Context template

See [references/template.md](references/template.md) for the exact Markdown structure.

## Gap handling

See [references/gap-handling.md](references/gap-handling.md) for how to record non-critical and blocking gaps.

## Doubts and clarification policy

This skill does **not** ask interactive questions during analysis.

All functional or technical doubts discovered during investigation are recorded as **gaps** in `.context/hu-<code>.md`.

The companion skill `bcv-dhu-writer` will present these gaps at the end of the DHU with **suggested answers**, so the user can respond in a single turn.

### Types of gaps

| Type | Definition | Example |
|---|---|---|
| `business` | Requires clarification from product/business. | Order of dropdown, catalog source. |
| `technical` | Requires clarification from architecture or service owner. | Field location, integration pattern. |
| `implementation` | Can be decided by the development team. | Exact JSON field name. |

### Rules

- Do not stop analysis to ask the user.
- Record every doubt as a gap with a concise description.
- Categorize each gap as `blocking` or `non-blocking`.
- For `business` gaps, include a suggested answer when possible.

## Example

See [references/example.md](references/example.md) for a complete walkthrough with the "Oficina Registral" HU.

## Optimizations

To minimize token usage in production, the skill relies on these optimizations:

1. **Pre-built graphify graphs** — repositories should have `graphify-out/graph.json` generated ahead of time (CI job or one-time build) so the skill only runs queries, not full extraction.
2. **Minimal workspace context** — a single `docs/.agent-context/service-map.md` with role, stack, and god nodes per service.
3. **Optional service-level gotchas** — a very short `<repo>/docs/.agent-context/gotchas.md` (max 10 lines) only for non-obvious constraints.
4. **`--code-only` graphify builds** — avoids LLM extraction of non-code files.

The skill does **not** rely on a per-service baseline cache. All reusable learning should fit in `service-map.md` or `gotchas.md`, or be rediscovered cheaply via graphify.

See [references/optimizations.md](references/optimizations.md) for implementation details.

## When to update graphify and which repo

### Rebuild triggers

| Condition                                               | Repo to update                               |
| ------------------------------------------------------- | -------------------------------------------- |
| New REST controller, subscriber, publisher, or use case | The service where the change occurs          |
| New database entity or table                            | The service that owns the entity             |
| New Feign client or external integration                | The service that consumes it                 |
| Rename/removal of a god node                            | The affected service                         |
| Significant package refactor                            | The affected service                         |
| Merge to `main` with structural changes                 | All affected services (prefer CI automation) |

### No rebuild needed

| Situation                        | Reason                  |
| -------------------------------- | ----------------------- |
| Bugfix inside existing method    | AST node already exists |
| Add field to existing DTO/entity | Node already exists     |
| New tests                        | Not in production graph |
| Docs-only changes                | Not in code graph       |

### Quick check

```bash
graphify check-update <repo-path>
```

For full details and CI examples, see [references/optimizations.md](references/optimizations.md) section 6.

## References

- [workflow.md](references/workflow.md)
- [limits.md](references/limits.md)
- [template.md](references/template.md)
- [gap-handling.md](references/gap-handling.md)
- [example.md](references/example.md)
- [optimizations.md](references/optimizations.md)
