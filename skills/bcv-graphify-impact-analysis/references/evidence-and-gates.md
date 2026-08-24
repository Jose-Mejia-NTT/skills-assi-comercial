# Evidence and gate policy for bcv-graphify-impact-analysis

## Purpose

Define how to grade technical evidence and determine the Step 2 gate status.

## Evidence priority

1. Current graph metadata + path/query evidence for candidate service.
2. Current source code symbols linked from Graphify results.
3. Service map dependency evidence.
4. README/context docs marked as current.
5. Draft/snapshot docs.

## Freshness rules

- If graph timestamp or code reference is missing, do not mark READY unless corroborated by source symbols.
- If most evidence is draft/snapshot, default to REVIEW_REQUIRED.
- If ownership cannot be defended with at least medium confidence, use REVIEW_REQUIRED or BLOCKED.

## Gate criteria

### READY

Use when:
- primary service has confirmed evidence;
- major impact dimensions (API, persistence, events) are either confirmed or explicitly not applicable;
- remaining assumptions do not block Step 3 drafting.

### REVIEW_REQUIRED

Use when:
- ownership ambiguity persists;
- contradictory evidence exists;
- key impacts are candidate-only;
- external dependency is required and not visible locally.

### BLOCKED

Use when:
- HU or candidate inputs are missing;
- no defensible impact evidence is available;
- analysis would require inventing APIs/events/persistence details.
