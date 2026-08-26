---
name: bcv-implementation-orchestrator
description: |
  Use this skill whenever the user wants to implement a BCV HU/HAB that already has an
  approved technical-story-enriched.md. The skill turns the story into an implementation
  orchestration plan by mapping each IMP-XXX task to the right BCV implementation skill
  (bcv-clean-architecture, bcv-hexagonal-architecture, bcv-java-spring-boot,
  bcv-openapi-design, bcv-spring-data-jpa-sql-server, bcv-cosmos-db,
  bcv-azure-service-bus, bcv-commons-observability) and produces ready-to-use prompts.
  Activate for: "implementar HU", "generar codigo", "ejecutar plan de implementacion",
  "orquestar implementacion", "story to code", "implementation orchestration",
  "pasar la historia tecnica a codigo", "implementar desde technical-story-enriched.md".
  Do NOT use for business resolution, technical impact analysis or test generation in isolation.
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "implementation-orchestration"
  frameworks: ["Spec-Driven Development", "BMAD"]
  argument-hint: "technical-story-enriched.md + technical-impact-analysis.yaml"
---

# bcv-implementation-orchestrator

## Objective

Transform an approved `technical-story-enriched.md` into an actionable implementation
orchestration plan. This skill does **not** write source code. It delegates each
implementation task to the appropriate BCV specialized skill and produces the exact
prompts those skills need.

This is **Step 3** of the pipeline:

```text
HU/HAB
  ↓
bcv-business-resolution
  → business-resolution.yaml
  ↓
bcv-technical-impact-and-story
  → technical-impact-analysis.yaml
  → technical-story-enriched.md
  ↓
bcv-implementation-orchestrator
  → implementation-orchestration-plan.md
  → implementation-prompts/<task-id>.md
```

## When to use

Activate this skill when the user asks to:

- implement a HU/HAB from an approved technical story;
- generate code from `technical-story-enriched.md`;
- execute the implementation plan;
- orchestrate the implementation tasks.

## When NOT to use

Do not activate for:

- business or functional resolution (use `bcv-business-resolution`);
- technical impact analysis (use `bcv-technical-impact-and-story`);
- isolated API design, test generation or database design without a story;
- writing code directly without delegating to specialized skills.

## Input expected

Preferred input:

1. `technical-implementation-blueprint.yaml` from `bcv-technical-impact-and-story`.

Fallback input (legacy):

1. `technical-story-enriched.md` from `bcv-technical-impact-and-story`.
2. `technical-impact-analysis.yaml` from `bcv-technical-impact-and-story`.

Additional inputs:

3. Optional confirmed stack: `java-spring-boot` (default), `nodejs`, etc.
4. Optional `available_skills` mapping for Copilot or other assistants.

Minimum to run without blocking:

- A `technical-implementation-blueprint.yaml` with numbered tasks (`IMP-XXX`), services, files and skill categories; or
- A `technical-story-enriched.md` with numbered tasks and a `technical-impact-analysis.yaml` with `technical_status`.

If the story contains `Status: BLOCKED` tasks, include them in the plan but mark them
as not executable until their unblock condition is met.

## Output location

Write artifacts to `docs/historial/` using the HU slug as prefix:

- `docs/historial/<hu-slug>-implementation-orchestration-plan.md`
- `docs/historial/<hu-slug>-implementation-prompts/` (one file per executable task)

## Workflow

### Phase A — Parse and validate

1. If `technical-implementation-blueprint.yaml` exists, load it as the primary source of truth.
   - Use `technical-story-enriched.md` and `technical-impact-analysis.yaml` only for additional context or when the blueprint is missing.
2. If using the blueprint, verify `technical_status` is `READY` or `REVIEW_REQUIRED` from its metadata.
3. If using story + impact analysis, load both and verify `technical_status`.
4. Extract all `IMP-XXX` tasks with:
   - type, description, step-by-step implementation, dependencies, status;
   - files affected with operation and change description;
   - DoR, DoD, acceptance criteria, test cases, error scenarios;
   - recommended skill category or mapped skill.
5. Build the dependency graph from `Dependencies` and `Unblock condition`.

### Phase B — Map tasks to specialized skills

Reference mapping: `references/implementation-task-to-skill-mapping.md`.

For each task with `Status: TODO`:

1. Read `analysis_scope.stack` from the impact analysis (default: `java-spring-boot`).
2. Determine the specialized skill based on task type and stack using
   `references/implementation-task-to-skill-mapping.md`.
3. If the user provided `available_skills` and the mapped skill is **not** in that list,
   mark the task as `MANUAL` instead.
4. For `TODO` tasks with a valid skill, generate a self-contained prompt containing:
   - the task context from the story;
   - the specific acceptance criteria;
   - the files affected;
   - the expected output files;
   - references to `technical-impact-analysis.yaml` when needed.

For each task with `Status: BLOCKED`:

1. Do **not** generate an implementation prompt.
2. Do **not** create a file under `implementation-prompts/`.
3. Record the unblock condition and the blocking task in the orchestration plan only.

### Phase C — Produce orchestration plan

1. Order tasks topologically using the dependency graph.
2. Group tasks by service or domain when possible.
3. Write the orchestration plan to disk.
4. Write one prompt file per executable task to `implementation-prompts/`.

## Rules and guardrails

1. **Do not write source code**. Only produce prompts and orchestration instructions.
2. **Do not execute BLOCKED tasks**. Keep them as pending with unblock conditions.
3. **Do not invent requirements**. Every prompt must derive from the story and impact analysis.
4. **Preserve trazability**: HU criterion → IMP-XXX task → specialized skill → expected output.
5. **Use the specialized BCV skills** for implementation concerns; do not replace them.
6. **If a task type has no matching skill**, document it as a manual task in the plan.
7. **Keep prompts self-contained**. A developer should be able to copy-paste them into the target skill.

## Language Handling and Output Policy

The skill must clearly separate **internal processing language** from **user-facing output language**.

### Language Detection

1. At the start of every interaction, detect the language of the user's input.
2. The **user-facing responses** must always be written in the user's language.
3. If the user's language cannot be confidently detected, default to English.

### Output Language (User-Facing)

All content exposed to the user must be written in the user's language, including:

- Explanations and confirmations
- Implementation orchestration plan
- Acceptance criteria
- Documentation intended for human readers

### Internal Processing Language

- For quality, consistency and technical accuracy, the skill may internally reason, plan, structure and code content in English.
- Internal reasoning language must never leak into the user-facing output.

### Technical Artifacts Language

To follow industry standards and best practices:

- **Source code**, **class names**, **method names**, **variable names** and **package names** must be written in English.
- **OpenAPI fields**, **JSON keys**, and **HTTP-level constructs** must be written in English.
- **Git commit messages** must be written in English unless the user explicitly requests otherwise.

### Important Clarification

The skill template and internal logic may be defined in English, but **all responses visible to the user must respect the detected user language**.

### Additional preservation rules

- Preserve status values, artifact paths, BCV names, business terms, service names and public identifiers exactly as supplied.
- Do not translate YAML keys or status values (`READY`, `REVIEW_REQUIRED`, `BLOCKED`, `TODO`).

## Token-efficiency rules

- Read only the sections of the story relevant to the current task.
- Do not reload the whole codebase for every task.
- Prefer structured fields over narrative repetition.
- Reuse the same context block in every prompt rather than repeating it.

## Completion checklist (Definition of Done)

1. Every `TODO` task has a mapped specialized skill and a self-contained prompt.
2. Every `BLOCKED` task is recorded with its unblock condition.
3. Task order respects dependencies.
4. The orchestration plan references the blueprint (or the story + impact analysis as fallback).
5. No source code is written by this skill.
6. No inference is needed: services, files and skill mappings come from the blueprint.
7. The user response is in the user's language.

## Activation quick check

Activate this skill when the request asks to:

- implement a HU from an approved technical story;
- orchestrate implementation tasks;
- generate ready-to-use prompts for BCV implementation skills.

## References

| File                                                   | Purpose                                            |
| ------------------------------------------------------ | -------------------------------------------------- |
| `assets/implementation-orchestration-plan.template.md` | Template for the orchestration plan output.        |
| `references/implementation-task-to-skill-mapping.md`   | Maps IMP-XXX task types to BCV specialized skills. |
| `references/orchestration-rules.md`                    | Detailed orchestration rules and guardrails.       |
