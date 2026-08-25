# Capability-resolution rules

1. Match by capability first; do not map an entity directly to a service without context.
2. Use entity, action, event and system combinations to rank matches.
3. Distinguish roles: `owner`, `orchestrator`, `participant`, `producer`, `consumer`, `data_owner`,
   `integration_adapter` and `possible_downstream`.
4. Include the catalog entry or business evidence supporting every candidate.
5. Never invent APIs, tables, topics, classes, packages or technical dependencies.
6. Do not claim that a candidate is the exact service. That decision belongs to technical impact analysis.
7. If multiple candidates remain plausible, return all material candidates and set `REVIEW_REQUIRED`.
8. If no candidate has sufficient evidence, set `BLOCKED` and state what catalog or business input is missing.
9. Never use an arbitrary confidence percentage. Use `high`, `medium`, `low` and explain the evidence.
10. Do not infer ownership from a generic entity such as `customer`, `account` or `email` alone.
11. Prefer a capability plus action and business context over an isolated entity match.
12. Preserve repository aliases exactly when the catalog or input provides them, including the
    `bcv-bacc-*` service prefix.
13. Record evidence source and date when available; stale, draft or snapshot sources lower confidence.
14. If catalog ownership conflicts with service-map context, set `REVIEW_REQUIRED` and expose the conflict.

## Rule categories

### Matching rules

- Rules 1, 2, 11: always match capability first, combining entity + action + event + system.

### Role assignment rules

- Rules 3, 10: distinguish orchestrator, data owner and downstream services; do not infer ownership from generic entities.

### Evidence rules

- Rules 4, 9, 13: every candidate needs provenance; use qualitative confidence and record source/date.

### Guardrail rules

- Rules 5, 6, 7, 8, 12, 14: never invent technical facts; preserve aliases; expose conflicts and uncertainty.

## Resolution status

- `CANDIDATES_FOUND`: at least one supported candidate exists and no material ambiguity blocks handoff.
- `REVIEW_REQUIRED`: multiple plausible candidates, conflicting ownership, missing business meaning or
  insufficient evidence prevents selecting a primary candidate.
- `BLOCKED`: the request is not sufficiently specified or no capability mapping can be supported.

A `CANDIDATES_FOUND` result still represents candidates, not implementation approval.
