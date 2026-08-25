# bcv-business-resolution

Business-discovery skill for the BCV HU/HAB flow, configured for GitHub Copilot.

## Purpose

Transforms a BCV HU, HAB or functional requirement into a normalized business context and an evidence-based list of business capabilities and candidate microservices.

This is **Step 1** of the pipeline:

```
HU/HAB → bcv-business-resolution → bcv-technical-impact-and-story → implementation-ready story
```

## When to use

Use this skill when the user asks to:

- analyze a HU/HAB
- extract business entities, actions or events
- identify business capabilities
- map business language to candidate services
- identify possible affected microservices **before** technical analysis

## Output

- `docs/historial/<hu-slug>-business-resolution.yaml`

The YAML is consumed by `bcv-technical-impact-and-story`.

## Key files

| File                                          | Purpose                                              |
| --------------------------------------------- | ---------------------------------------------------- |
| `SKILL.md`                                    | Main skill instructions (executive index).           |
| `assets/business-resolution.template.yaml`    | Output contract template.                            |
| `assets/examples.md`                          | Input/output examples.                               |
| `references/bcv-bacc-capability-catalog.yaml` | BACC pilot capability catalog.                       |
| `references/evidence-and-resolution.md`       | Evidence hierarchy and resolution gates.             |
| `references/extraction-rules.md`              | Rules for extracting business concepts.              |
| `references/capability-resolution-rules.md`   | Rules for matching capabilities and assigning roles. |
| `references/guardrails.md`                    | Restrictions and safety rules.                       |
| `references/evaluation-criteria.md`           | Evaluation criteria, DoD and activation rules.       |

## Pipeline handoff

The output artifact is the input for `bcv-technical-impact-and-story`.
The `handoff` section of the YAML contains:

- `to_skill`: `bcv-technical-impact-and-story`
- `output_artifact`: path to the written YAML
- `business_resolution_status`: `CANDIDATES_FOUND`, `REVIEW_REQUIRED` or `BLOCKED`
- `original_request_summary`
- `acceptance_criteria`
- `candidate_verification_questions`
- `technical_verification_questions`
- `assumptions` and `risks`
- candidates with roles, confidence and evidence
- ambiguities

## What this skill does NOT do

- Technical impact analysis
- API design
- Implementation or source code changes
- Test generation

These belong to `bcv-technical-impact-and-story` or downstream skills.
