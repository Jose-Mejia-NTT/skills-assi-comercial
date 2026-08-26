# bcv-implementation-orchestrator

Implementation-orchestration skill for the BCV HU/HAB flow, configured for GitHub Copilot.

## Purpose

Turns an approved technical story into an implementation orchestration plan that
delegates each task to the right BCV specialized skill. This skill does **not**
write source code.

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

Use this skill when the user asks to:

- implement a HU from an approved technical story;
- generate code from `technical-story-enriched.md`;
- execute the implementation plan;
- orchestrate implementation tasks.

## Output

- `docs/historial/<hu-slug>-implementation-orchestration-plan.md`
- `docs/historial/<hu-slug>-implementation-prompts/` (one prompt per executable task)

## Key files

| File | Purpose |
|---|---|
| `SKILL.md` | Main skill instructions. |
| `assets/implementation-orchestration-plan.template.md` | Template for the orchestration plan. |
| `references/implementation-task-to-skill-mapping.md` | Maps task types to specialized BCV skills. |
| `references/orchestration-rules.md` | Detailed orchestration rules and guardrails. |

## Pipeline input

This skill consumes the output of `bcv-technical-impact-and-story`:

- `docs/historial/<hu-slug>-technical-story-enriched.md`
- `docs/historial/<hu-slug>-technical-impact-analysis.yaml`

## Specialized BCV implementation skills referenced

- `bcv-clean-architecture`
- `bcv-hexagonal-architecture`
- `bcv-java-spring-boot`
- `bcv-openapi-design`
- `bcv-spring-data-jpa-sql-server`
- `bcv-cosmos-db`
- `bcv-azure-service-bus`
- `bcv-commons-observability`

## Validation

This skill was validated against 5 eval cases covering:

1. Multiple available BCV skills with BLOCKED dependencies.
2. No available skills (generic categories).
3. Ambiguity-driven BLOCKED tasks.
4. Non-Java stack with missing backend skill.
5. All TODO tasks with full skill coverage.

See `evals/evals.json` for details and `../bcv-implementation-orchestrator-workspace/iteration-1/` for sample runs.

## Distribution

Packaged skill:

```text
/Users/joseluis/.agents/skills/skill-creator/bcv-implementation-orchestrator.skill
```

Use the workspace script to repackage:

```bash
python3 package-skill.py bcv-implementation-orchestrator --output-dir "/Users/joseluis/.agents/skills/skill-creator" --verbose
```

## What this skill does NOT do

- Business or functional resolution.
- Technical impact analysis.
- Write source code directly.
- Generate tests in isolation.
