# bcv-technical-impact-and-story

Technical-analysis skill for the BCV HU/HAB flow, configured for GitHub Copilot.

## Purpose

Turns a validated business resolution into linked artifacts:

1. `technical-impact-analysis.yaml` — evidence-graded technical impact matrix plus a gate status.
2. `technical-story-enriched.md` — implementation-ready technical story with numbered, traceable tasks,
   architecture diagram, detailed action plan (step-by-step implementation, DoR, DoD, test cases,
   error scenarios, deployment notes, files affected), repository file impact and developer review & sign-off.
3. `technical-architecture-diagram.mmd` — Mermaid source of the implementation architecture.

This is **Step 2** of the pipeline:

```
HU/HAB → bcv-business-resolution → bcv-technical-impact-and-story → implementation-ready story
```

## When to use

Use this skill when the user asks to:

- validate business-resolution candidates against graph and repository evidence
- identify impacted APIs, persistence or events
- emit a `READY` / `REVIEW_REQUIRED` / `BLOCKED` technical gate
- produce an implementation-ready technical story from a validated impact analysis

## Execution modes

| Mode          | Condition                                                    | Output          |
| ------------- | ------------------------------------------------------------ | --------------- |
| `full`        | HU + `business-resolution.yaml`, no existing impact analysis | `.yaml` + `.md` |
| `story-only`  | existing `technical-impact-analysis.yaml`                    | `.md` only      |
| `impact-only` | user asks only for impact analysis                           | `.yaml` only    |

## Output

- `docs/historial/<hu-slug>-technical-impact-analysis.yaml`
- `docs/historial/<hu-slug>-technical-story-enriched.md`
- `docs/historial/<hu-slug>-technical-architecture-diagram.mmd`

## Key files

| File                                             | Purpose                                            |
| ------------------------------------------------ | -------------------------------------------------- |
| `SKILL.md`                                       | Main skill instructions (executive index).         |
| `assets/technical-impact-analysis.template.yaml` | Template for the impact analysis YAML.             |
| `assets/technical-story-enriched.template.md`    | Template for the enriched technical story.         |
| `assets/examples.md`                             | Concrete examples for all execution modes.         |
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
