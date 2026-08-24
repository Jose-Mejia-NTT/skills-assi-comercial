# Evidence and gate policy

## Purpose

Define how to grade technical evidence and determine the gate status emitted by Phase A.

## Evidence priority

1. Current graph metadata + path/query evidence for the candidate service.
2. Current source code symbols linked from graph results.
3. Service map dependency evidence.
4. README/context docs marked as current.
5. Draft/snapshot docs.

## Freshness rules

- If the graph timestamp or code reference is missing, do not mark `READY` unless corroborated by source symbols.
- If most evidence is draft/snapshot, default to `REVIEW_REQUIRED`.
- If ownership cannot be defended with at least medium confidence, use `REVIEW_REQUIRED` or `BLOCKED`.

## Gate criteria

### READY

Use when:

- the primary service has confirmed evidence;
- major impact dimensions (API, persistence, events) are either confirmed or explicitly not applicable;
- remaining assumptions do not block story drafting.

### REVIEW_REQUIRED

Use when:

- ownership ambiguity persists;
- contradictory evidence exists;
- key impacts are candidate-only;
- an external dependency is required and is not visible locally.

### BLOCKED

Use when:

- HU or candidate inputs are missing;
- no defensible impact evidence is available;
- the analysis would require inventing APIs, events or persistence details.

## Gate barrier consequences

The gate is not advisory. It controls whether Phase B runs at all.

| Status | Phase B | Story requirement |
| --- | --- | --- |
| `READY` | runs | full story |
| `REVIEW_REQUIRED` | runs | must open with a "Pending architecture review" block listing every conflict and candidate-only impact |
| `BLOCKED` | does not run | no story is produced; deliver the YAML and the missing prerequisites |

Never upgrade a status to unblock Phase B. If the gate is uncomfortable, fix the evidence, not the status.
