---
name: bcv-business-resolution
description: |
  Use this skill when the user provides a BCV user story, HAB, HU, business requirement
  or functional request and needs its business concepts mapped to business capabilities
  and candidate microservices. Triggers: "analizar HU", "analizar HAB", "extraer capacidades",
  "identificar servicios afectados", "microservicios candidatos", "resolver capacidad de negocio",
  "enriquecer requerimiento". Do NOT use for Graphify architectural impact analysis,
  technical story generation, API design, implementation, database changes, messaging changes,
  or test generation.
argument-hint: "HU/HAB + optional capability catalog"
metadata:
  version: "1.1.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "business-discovery"
  frameworks: ["Spec-Driven Development", "BMAD"]
---

# bcv-business-resolution

## Objective

Transform a BCV HU, HAB or functional requirement into a normalized business context and
an evidence-based list of business capabilities and candidate microservices.

This skill reduces the technical search space. It does not select a final implementation
service, query Graphify, inspect source code, generate a technical impact analysis, or modify files.

## Mandatory BCV context bootstrap

Before resolving any BCV HU/HAB, load context in this strict order:

1. `docs/.agent-context/graph-index.md`
2. `docs/.agent-context/service-map.md`
3. `docs/.agent-context/graphify-query-playbook.md`
4. `docs/cross-service-patterns.md` when available; otherwise use `docs/.agent-context/cross-service-patterns.md`.

Purpose of each source:

- `graph-index.md`: fast capability-to-service orientation and relevant concept hotspots.
- `service-map.md`: ownership boundaries, dependencies and cross-service interactions.
- `graphify-query-playbook.md`: handoff contract for the next technical phase and query style.
- `cross-service-patterns.md`: shared cross-service vocabulary and interaction patterns used only
  to standardize wording in handoff (no technical confirmation in Step 1).

This bootstrap is mandatory for BCV workspace consistency. It does not authorize Graphify
queries in this skill; it only grounds business resolution with shared context.

## Input expected

Provide one or more of:

1. HU, HAB, business requirement or functional request.
2. Acceptance criteria and business rules.
3. Optional capability catalog, aliases, ownership and relationship mappings.
4. Optional domain glossary or known business systems.

If no catalog is available, extract concepts and report that service resolution is provisional.

Minimum required input to run without blocking:

1. Functional request text (HU/HAB or equivalent).
2. At least one of: acceptance criteria, explicit rule, or in-scope/out-of-scope statement.

If these minimums are missing, return `BLOCKED` with targeted questions.

For the BCV BACC pilot, load only when relevant:

- `references/bcv-bacc-capability-catalog.yaml` for the initial capability and service map.
- `references/evidence-and-resolution.md` for evidence ranking, roles and resolution gates.

Treat the pilot catalog as versioned working knowledge, not as a final ownership decision.

This skill implements only **Step 1: Functional Resolution** from the HU/HAB to HT-enriched flow.
Step 2 (technical impact) and Step 3 (technical story enrichment) are downstream.

## Output expected

Return a concise structured result with these sections:

1. `business_context`: systems, entities, actions, events, rules and acceptance criteria.
2. `capabilities`: normalized capabilities matched from the request.
3. `candidates`: possible services with role, confidence and evidence.
4. `ambiguities`: unresolved business meanings or missing information.
5. `resolution_status`: `CANDIDATES_FOUND`, `REVIEW_REQUIRED` or `BLOCKED`.
6. `handoff`: exact questions and inputs required by the technical impact-analysis phase.

Minimum handoff content:

- original functional request summary;
- acceptance criteria and explicit in-scope/out-of-scope statements;
- candidate list with roles (`owner`, `orchestrator`, `data_owner`, `participant`, `possible_downstream`);
- confidence (`high|medium|low`) and evidence provenance per candidate;
- ambiguities, assumptions (`ASSUMED`) and risks (`RISK`);
- concrete verification questions for technical impact analysis.

Use YAML when the result will be consumed by another skill. Use Markdown with the same fields
when the user asks for a human-readable analysis.

Default output artifact name: `business-resolution.yaml`.
Reference template: `assets/business-resolution.template.yaml`.

Output contract (required top-level keys):

- `business_context`
- `capabilities`
- `candidates`
- `ambiguities`
- `resolution_status`
- `handoff`

Any additional key must be optional and justified by explicit user need.

## Workflow

### SDD

1. **Specify**: parse the HU/HAB into a structured business specification.
2. **Validate**: check actors, goals, entities, actions, events, rules, acceptance criteria and scope.
3. **Generate**: normalize capabilities and resolve candidates from the available catalog.
4. **Verify**: check traceability, ambiguity handling and output status.

### Functional-resolution execution steps

1. Extract business concepts: systems, entities, actions, events, rules and restrictions.
2. Normalize business terms and aliases without losing critical domain meaning.
3. Resolve capabilities and propose a small candidate set with explicit roles.
4. Assign qualitative confidence with evidence rationale.
5. Emit `resolution_status`: `CANDIDATES_FOUND`, `REVIEW_REQUIRED`, or `BLOCKED`.

### Decision branching

1. `CANDIDATES_FOUND`: there is enough functional evidence to hand off candidates.
2. `REVIEW_REQUIRED`: ambiguity or conflict exists but analysis can still be handed off with warnings.
3. `BLOCKED`: minimum inputs are missing or no capability mapping can be supported.

### BMAD

1. **Understand**: identify the business goal, actors, entities, actions, systems and constraints.
2. **Design**: define normalized capabilities, aliases and candidate service roles.
3. **Build**: produce the structured business-resolution output.
4. **Validate**: confirm that every candidate has evidence and that uncertain results are gated.

## Extraction rules

Extract only concepts supported by the input or catalog:

- `systems`: named business or external systems.
- `entities`: business objects, roles and important data elements.
- `actions`: business verbs and state transitions.
- `events`: business events, triggers and outcomes.
- `rules`: thresholds, eligibility, exclusions, defaults and invariants.
- `scope`: explicit inclusions and exclusions.

Normalize spelling and aliases without changing business meaning. Preserve original terms when
normalization could lose meaning.

## Capability-resolution rules

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

## Resolution status

- `CANDIDATES_FOUND`: at least one supported candidate exists and no material ambiguity blocks handoff.
- `REVIEW_REQUIRED`: multiple plausible candidates, conflicting ownership, missing business meaning or
  insufficient evidence prevents selecting a primary candidate.
- `BLOCKED`: the request is not sufficiently specified or no capability mapping can be supported.

A `CANDIDATES_FOUND` result still represents candidates, not implementation approval.

## Handoff contract

The next phase, `bcv-graphify-impact-analysis`, should receive:

- the original HU/HAB;
- this business-resolution result;
- candidate service names and roles;
- evidence and confidence;
- ambiguities and unresolved questions;
- the requested scope and acceptance criteria.

The next phase must validate candidates against Graphify and current repository evidence before
identifying affected APIs, persistence, events, dependencies or implementation files.

For the BACC pilot, include the source snapshot or commit when it is available. Documents marked
as draft, snapshot or `NEEDS_REVIEW` must lower confidence or produce `REVIEW_REQUIRED`.

If the request remains ambiguous after extraction, ask up to 3 targeted clarification questions that can
change candidate resolution. Otherwise, proceed and mark uncertainty explicitly.

## Guardrails

1. Do not inspect or modify source files as part of this skill.
2. Do not query `GRAPH_REPORT.md`, `graph.json`, `graph.html` or Graphify MCP.
3. Do not generate a technical story, implementation plan or production code.
4. Do not hide missing catalog entries or unsupported terms.
5. Mark assumptions as `ASSUMED` and risks as `RISK`.
6. Ask up to 3 focused clarification questions only when they can change the candidate mapping.
7. Keep the output concise and suitable for handoff to another skill.
8. Never output Step 2 conclusions (`impacted_apis`, `impacted_persistence`, `impacted_events`) as facts.
9. Do not assert synchronous/asynchronous integration paths unless they are explicitly in provided
  business input or reference catalog.

## Language rules

- Internal processing, structural reasoning and generated technical artifacts: English.
- Response to the user: the language of the user's initial message; default to Spanish.
- Preserve BCV names, business terms, service names and public identifiers exactly when supplied.

## Token-efficiency rules

- Read only the HU/HAB and the relevant catalog sections.
- Do not load an entire repository or unrelated capability mappings.
- Prefer structured fields and short evidence statements over narrative repetition.
- Do not repeat the original HU when a normalized field captures the same information.
- Return only material candidates and ambiguities.

## Evaluation

The output is valid when it:

1. Separates business concepts from technical candidates.
2. Extracts entities, actions, events, rules and scope without inventing facts.
3. Uses the capability catalog when available and reports missing mappings.
4. Assigns candidate roles and explainable qualitative confidence.
5. Preserves multiple candidates when ownership is ambiguous.
6. Uses the correct resolution status.
7. Produces a complete handoff for technical impact analysis.
8. Does not query Graphify or claim technical evidence it did not inspect.
9. Uses the BACC reference catalog consistently when the request belongs to the pilot scope.
10. Distinguishes an orchestrator from a data owner and a downstream reporting service.
11. Shows evidence provenance and resolution rationale per candidate.
12. Reflects the mandatory BCV context bootstrap order before candidate resolution.

## Completion checklist (Definition of Done)

1. All six required output keys are present.
2. Every candidate has role, confidence and at least one evidence source.
3. At least one ambiguity or explicit "none" statement is provided.
4. `resolution_status` is consistent with ambiguity and evidence strength.
5. Handoff includes concrete verification questions for technical impact analysis.
6. No Step 2 or Step 3 facts are presented as confirmed outcomes.

## Activation quick check

Activate this skill when the request contains a HU, HAB or functional requirement and asks to:

- extract business entities, actions or events;
- identify capabilities;
- map business language to candidate services;
- identify possible affected microservices before technical analysis.
