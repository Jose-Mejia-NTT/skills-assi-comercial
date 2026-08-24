# Quality checks for the enriched technical story

## Purpose

Deterministic checks before declaring the technical story ready for refinement.

## Required checks

1. Evidence isolation
- Every technical item in the story exists in the emitted `technical-impact-analysis.yaml`.
- No item was added from Phase A working memory without being recorded in the YAML first.
- The story states the YAML path and its `technical_status`.

2. Scope coherence
- Primary and supporting services in the story match the YAML.
- Any deviation is explicitly justified as a review item.

3. Impact integrity
- APIs, persistence and events preserve the confirmed vs candidate split.
- No technical item appears without an upstream evidence reference.

4. Task coverage
- The task plan includes contract, domain/application, persistence/messaging,
  observability/security and testing.
- Each task links to at least one HU criterion and one impact item.

5. Risk and assumption clarity
- Every unresolved dependency is either `ASSUMED` or `RISK`.
- A `REVIEW_REQUIRED` gate produced explicit pending validation items.

6. Acceptance quality
- The checklist includes coverage, non-regression and traceability checks.
- Criteria are verifiable, not generic placeholders.

## Escalation rules

- Gate `BLOCKED`: do not emit a story.
- Gate `REVIEW_REQUIRED`: emit a draft with the pending-review block at the top and unresolved items listed.
- Evidence stale or contradictory: keep `REVIEW_REQUIRED` and escalate to technical review.
