# Evaluation criteria

The output is valid when it:

1. Separates business concepts from technical candidates.
2. Extracts entities, actions, events, rules and scope without inventing facts.
3. Uses the capability catalog when available and reports missing mappings.
4. Assigns candidate roles and explainable qualitative confidence.
5. Preserves multiple candidates when ownership is ambiguous.
6. Uses the correct resolution status.
7. Produces a complete handoff for technical impact analysis.
8. Does not read graph artifacts or claim technical evidence it did not inspect.
9. Uses the BACC reference catalog consistently when the request belongs to the pilot scope.
10. Distinguishes an orchestrator from a data owner and a downstream reporting service.
11. Shows evidence provenance and resolution rationale per candidate.
12. Reflects the mandatory BCV business context bootstrap order before candidate resolution.

## Completion checklist (Definition of Done)

1. All six required output keys are present.
2. Every candidate has role, confidence and at least one evidence source.
3. Every ambiguity has `status`, `blocking`, `owner` and `resolution` when not `pending`.
4. No `resolution_status` is `CANDIDATES_FOUND` or `REVIEW_REQUIRED` if any ambiguity has `status: pending` and `blocking: true`.
5. `resolution_status` is consistent with ambiguity and evidence strength.
5. Handoff includes concrete verification questions for technical impact analysis.
6. No Step 2 or Step 3 facts are presented as confirmed outcomes.

## Activation quick check

Activate this skill when the request contains a HU, HAB or functional requirement and asks to:

- extract business entities, actions or events;
- identify capabilities;
- map business language to candidate services;
- identify possible affected microservices before technical analysis.

## When NOT to activate

Do not activate for:

- Step 2 technical impact analysis;
- technical story generation;
- API design;
- implementation, database or messaging changes;
- test generation.
