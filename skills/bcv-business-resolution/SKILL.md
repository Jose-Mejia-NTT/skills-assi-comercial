---
name: bcv-business-resolution
description: |
  Use this skill when the user provides a BCV user story, HAB, HU, business requirement
  or functional request and needs its business concepts mapped to business capabilities
  and candidate microservices. Triggers: "analizar HU", "analizar HAB", "extraer capacidades",
  "identificar servicios afectados", "microservicios candidatos", "resolver capacidad de negocio",
  "enriquecer requerimiento". Do NOT use for Step 2 technical impact analysis,
  technical story generation, API design, implementation, database changes, messaging changes,
  or test generation.
argument-hint: "HU/HAB + optional capability catalog"
metadata:
  version: "1.2.0"
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
service, read architecture graph artifacts, inspect source code, generate a technical impact
analysis, or modify source code.

For detailed rules, examples and evaluation criteria, see the [References](#references) section.

## When to use / When NOT to use

Activate this skill when the request contains a HU, HAB or functional requirement and asks to:

- extract business entities, actions or events;
- identify capabilities;
- map business language to candidate services;
- identify possible affected microservices **before** technical analysis.

Do **NOT** use for:

- Step 2 technical impact analysis;
- technical story generation;
- API design;
- implementation, database or messaging changes;
- test generation.

See `references/evaluation-criteria.md` for the full activation checklist.

## Mandatory BCV business context bootstrap

Before resolving any BCV HU/HAB, load context in this strict order:

1. `references/bcv-bacc-capability-catalog.yaml` when the request belongs to the BACC pilot.
2. `docs/.agent-context/graph-index.md` — read only the service and capability columns.
3. `docs/.agent-context/service-map.md`
4. `docs/cross-service-patterns.md` when available; otherwise use `docs/.agent-context/cross-service-patterns.md`.

Purpose of each source:

- `bcv-bacc-capability-catalog.yaml`: authoritative capability-to-service ownership for the pilot.
- `graph-index.md`: fast capability-to-service orientation. Use it as a curated capability index only;
  ignore the node columns and never follow it into `graphify-out/` artifacts.
- `service-map.md`: ownership boundaries, dependencies and cross-service interactions.
- `cross-service-patterns.md`: shared cross-service vocabulary and interaction patterns used only
  to standardize wording in handoff (no technical confirmation in Step 1).

These are curated business-context documents. This bootstrap grounds candidate resolution with
shared vocabulary and ownership; it never authorizes reading graph reports or running graph
queries, which belong exclusively to Step 2.

For evidence ranking, roles and resolution gates, also load `references/evidence-and-resolution.md`.

## Input expected

Provide one or more of:

1. HU, HAB, business requirement or functional request.
2. Acceptance criteria and business rules.
3. Optional capability catalog, aliases, ownership and relationship mappings.
4. Optional domain glossary or known business systems.

Minimum required input to run without blocking:

1. Functional request text (HU/HAB or equivalent).
2. At least one of: acceptance criteria, explicit rule, or in-scope/out-of-scope statement.

If these minimums are missing, return `BLOCKED` with targeted questions.

This skill implements only **Step 1: Functional Resolution**. Step 2 (technical impact) and
Step 3 (technical story enrichment) are downstream.

## Output expected

Return a concise structured result with these required top-level keys:

1. `business_context`: systems, entities, actions, events, rules, acceptance criteria and scope.
2. `capabilities`: normalized capabilities matched from the request.
3. `candidates`: possible services with role, confidence and evidence.
4. `ambiguities`: unresolved business meanings or missing information.
5. `resolution_status`: `CANDIDATES_FOUND`, `REVIEW_REQUIRED` or `BLOCKED`.
6. `handoff`: exact questions and inputs required by the technical impact-analysis phase.

Use YAML when the result will be consumed by another skill. Use Markdown with the same fields
when the user asks for a human-readable analysis.

Default output artifact: `docs/historial/<hu-slug>-business-resolution.yaml`.
Reference template: `assets/business-resolution.template.yaml`.
Write the artifact automatically before responding and report its path.

Minimum handoff content:

- original functional request summary;
- acceptance criteria and explicit in-scope/out-of-scope statements;
- candidate list with roles (`owner`, `orchestrator`, `data_owner`, `participant`, `possible_downstream`);
- confidence (`high|medium|low`) and evidence provenance per candidate;
- ambiguities, assumptions (`ASSUMED`) and risks (`RISK`);
- concrete verification questions for technical impact analysis.

Any additional key must be optional and justified by explicit user need.

See `assets/examples.md` for concrete input/output examples.

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
3. **Build**: write the structured business-resolution YAML artifact.
4. **Validate**: confirm that every candidate has evidence and that uncertain results are gated.

## Handoff contract

The next skill, `bcv-technical-impact-and-story`, consumes this `business-resolution.yaml`.
The handoff must include:

- `to_skill`: `bcv-technical-impact-and-story`
- `output_artifact`: path to the written `business-resolution.yaml`
- `business_resolution_status`: `CANDIDATES_FOUND`, `REVIEW_REQUIRED` or `BLOCKED`
- `original_request_summary`: HU/HAB summary
- `acceptance_criteria`: explicit in-scope/out-of-scope statements
- `candidate_verification_questions`: questions that can change the candidate mapping
- `technical_verification_questions`: questions the technical phase must answer
- `assumptions` (prefixed `ASSUMED:`) and `risks` (prefixed `RISK:`)
- candidate service names, roles, confidence and evidence
- ambiguities and unresolved questions

The next phase must validate candidates against architecture graph artifacts and current
repository evidence before identifying affected APIs, persistence, events, dependencies or
implementation files.

For the BACC pilot, include the source snapshot or commit when it is available. Documents marked
as draft, snapshot or `NEEDS_REVIEW` must lower confidence or produce `REVIEW_REQUIRED`.

If the request remains ambiguous after extraction, ask up to 3 targeted clarification questions that can
change candidate resolution. Otherwise, proceed and mark uncertainty explicitly.

## Language Handling and Output Policy

The skill must clearly separate **internal processing language** from **user-facing output language**.

### Language Detection

1. At the start of every interaction, detect the language of the user's input.
2. The **user-facing responses** must always be written in the user's language.
3. If the user's language cannot be confidently detected, default to English.

### Output Language (User-Facing)

All content exposed to the user must be written in the user's language, including:

- Explanations and confirmations
- Functional specifications
- Acceptance criteria
- Error messages visible to consumers
- Documentation intended for human readers
- User-facing validation or confirmation messages

### Internal Processing Language

- For quality, consistency and technical accuracy, the skill may internally reason, plan, structure and code content in English.
- Internal reasoning language must never leak into the user-facing output.

### Technical Artifacts Language

To follow industry standards and best practices:

- **Source code**, **class names**, **method names**, **variable names** and **package names** must be written in English.
- **OpenAPI fields**, **JSON keys**, and **HTTP-level constructs** must be written in English.
- **Git commit messages** must be written in English unless the user explicitly requests otherwise.

### Important Clarification

The skill template and internal logic may be defined in English, but **all responses visible to the user must respect the detected user language**.

### Additional preservation rules

- Preserve status values, artifact paths, BCV names, business terms, service names and public identifiers exactly as supplied.
- Do not translate YAML keys or status values (`CANDIDATES_FOUND`, `REVIEW_REQUIRED`, `BLOCKED`).

## Token-efficiency rules

- Read only the HU/HAB and the relevant catalog sections.
- Do not load an entire repository or unrelated capability mappings.
- Prefer structured fields and short evidence statements over narrative repetition.
- Do not repeat the original HU when a normalized field captures the same information.
- Return only material candidates and ambiguities.

## Guardrails (summary)

- Do not inspect or modify source files as part of this skill.
- Do not read anything under `graphify-out/` or run graph queries. Graph evidence belongs to Step 2.
- Do not generate a technical story, implementation plan or production code.
- Do not hide missing catalog entries or unsupported terms.
- Mark assumptions as `ASSUMED` and risks as `RISK`.
- Ask up to 3 focused clarification questions only when they can change the candidate mapping.
- Keep the output concise and suitable for handoff to another skill.
- Never output Step 2 conclusions (`impacted_apis`, `impacted_persistence`, `impacted_events`) as facts.
- Do not assert synchronous/asynchronous integration paths unless explicitly in provided business input or reference catalog.

See `references/guardrails.md` for the full list.

## References

| File                                          | Purpose                                                                             |
| --------------------------------------------- | ----------------------------------------------------------------------------------- |
| `assets/business-resolution.template.yaml`    | Output contract template with all required keys.                                    |
| `assets/examples.md`                          | Concrete input/output examples, including a `REVIEW_REQUIRED` and a `BLOCKED` case. |
| `references/bcv-bacc-capability-catalog.yaml` | Authoritative capability-to-service ownership for the BACC pilot.                   |
| `references/evidence-and-resolution.md`       | Evidence hierarchy, candidate roles, confidence rules and discrepancy handling.     |
| `references/extraction-rules.md`              | Rules and patterns for extracting business concepts from HU/HAB.                    |
| `references/capability-resolution-rules.md`   | Detailed rules for matching capabilities, assigning roles and resolving status.     |
| `references/guardrails.md`                    | Full list of restrictions and safety rules.                                         |
| `references/evaluation-criteria.md`           | Output evaluation criteria, completion checklist and activation quick check.        |
