# Quality checks for bcv-technical-story-enricher

## Purpose

Provide deterministic checks before declaring the technical story ready for refinement.

## Required checks

1. Scope coherence
- Primary/supporting services in Step 3 must match Step 2 output.
- Any deviation must be explicitly justified as a review item.

2. Impact integrity
- APIs, persistence and events must preserve confirmed vs candidate split.
- No technical item may appear without upstream evidence reference.

3. Task coverage
- Task plan must include: contract, domain/application, persistence/messaging, observability/security, testing.
- Each task must link to at least one HU criterion and one impact item.

4. Risk and assumption clarity
- Every unresolved dependency must be either ASSUMED or RISK.
- REVIEW_REQUIRED inputs must produce explicit pending validation items.

5. Acceptance quality
- Checklist must include coverage, non-regression and traceability checks.
- Criteria must be verifiable and not generic placeholders.

## Escalation rules

- If Step 2 is BLOCKED: do not emit Step 3 story.
- If Step 2 is REVIEW_REQUIRED: emit Step 3 draft with review gates and unresolved items.
- If evidence is stale or contradictory: keep REVIEW_REQUIRED and escalate to technical review.
