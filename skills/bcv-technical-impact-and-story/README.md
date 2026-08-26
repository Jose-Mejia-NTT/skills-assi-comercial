# bcv-technical-impact-and-story

Technical-analysis skill for the BCV HU/HAB flow, configured for GitHub Copilot.

## Purpose

Turns a validated business resolution into linked artifacts:

1. `technical-impact-analysis.yaml` — evidence-graded technical impact matrix plus a gate status.
2. `technical-story-enriched.md` — implementation-ready technical story with numbered, traceable tasks,
   architecture diagram, detailed action plan (step-by-step implementation, DoR, DoD, test cases,
   error scenarios, deployment notes, files affected with operation and change description),
   repository file impact, developer review & sign-off and open questions / clarifications needed.
   When ambiguities exist, the story starts with `IMP-000` containing clarification questions and
   suggested resolution options so the resolver can decide quickly.
3. `technical-architecture-diagram.mmd` — Mermaid source of the implementation architecture.
4. `technical-implementation-blueprint.yaml` — machine-readable contract for `bcv-implementation-orchestrator`.
5. `technical-hu-document.md` — final technical HU document aligned with the project HU guideline.

This is **Step 2** of the pipeline:

```
HU/HAB → bcv-business-resolution → bcv-technical-impact-and-story → bcv-implementation-orchestrator
```

## When to use

Use this skill when the user asks to:

- validate business-resolution candidates against graph and repository evidence
- identify impacted APIs, persistence or events
- emit a `READY` / `REVIEW_REQUIRED` / `BLOCKED` technical gate
- produce an implementation-ready technical story from a validated impact analysis

## Execution modes

| Mode             | Condition                                                    | Output                                           |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| `full`           | HU + `business-resolution.yaml`, no existing impact analysis | `.yaml` + `.md` + `.mmd` + blueprint + doc-final |
| `story-only`     | existing `technical-impact-analysis.yaml`                    | `.md` + `.mmd` + blueprint + doc-final           |
| `impact-only`    | user asks only for impact analysis                           | `.yaml` only                                      |
| `blueprint-only` | existing story + impact analysis                             | blueprint only                                    |
| `document-only`  | existing story + impact analysis + blueprint                 | doc-final only                                    |

## Output

- `docs/historial/<hu-slug>-technical-impact-analysis.yaml`
- `docs/historial/<hu-slug>-technical-story-enriched.md`
- `docs/historial/<hu-slug>-technical-architecture-diagram.mmd`
- `docs/historial/<hu-slug>-technical-implementation-blueprint.yaml` (machine-readable contract for `bcv-implementation-orchestrator`)
- `docs/historial/<hu-slug>-technical-hu-document.md` (final technical HU document aligned with the project guideline)

## Key files

| File                                             | Purpose                                            |
| ------------------------------------------------ | -------------------------------------------------- |
| `SKILL.md`                                       | Main skill instructions (executive index).         |
| `assets/technical-impact-analysis.template.yaml` | Template for the impact analysis YAML.             |
| `assets/technical-story-enriched.template.md`    | Template for the enriched technical story.         |
| `assets/examples.md`                             | Concrete examples for all execution modes.         |
| `assets/technical-implementation-blueprint.template.yaml` | Machine-readable contract for the orchestrator. |
| `assets/technical-hu-document.template.md`       | Final technical HU document aligned with the project guideline. |
| `references/hu-document-guideline.md`            | Project standard for HU documents.                 |
| `references/hu-document-example.md`              | Real project DHU example.                          |
| `references/evidence-and-gates.md`               | Evidence grading and gate criteria.                |
| `references/graphify-query-discipline.md`        | Query budgets and escalation order (Phase A only). |
| `references/ambiguity-and-conflict-handling.md`  | How to handle inherited ambiguities and technical conflicts. |
| `references/skill-mapping.md`                    | Maps task types to skill categories; user maps categories to their own skills. |
| `references/guardrails.md`                       | Restrictions and safety rules.                     |
| `references/evaluation-criteria.md`              | Quality checks, DoD and activation rules.          |

## Pipeline input

This skill consumes the output of `bcv-business-resolution`:

- `docs/historial/<hu-slug>-business-resolution.yaml`

For Copilot, the user should also provide their available skills via:

- `available_skills`: a manual list in the prompt.
- `user_skills_path`: a path to a directory containing the user's skills (workspace-relative is recommended).

If neither is provided, the plan outputs skill categories and the user maps them later.

The impact analysis records:

- `analysis_scope.source_business_resolution`: path to the consumed YAML
- `analysis_scope.business_resolution_status`: status inherited from business resolution

## What this skill does NOT do

- Business or functional resolution (use `bcv-business-resolution`)
- Implementation or source code changes
- API design in isolation
- Test generation

The generated story contains actionable tasks but no estimates, assignments or code snippets.
