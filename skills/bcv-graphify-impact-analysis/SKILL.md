---
name: bcv-graphify-impact-analysis
description: |
  Use this skill when the user needs Step 2 technical impact analysis for a BCV HU/HAB,
  starting from business-resolution output and validating candidates with Graphify and
  repository evidence. Triggers: "analizar impacto tecnico", "validar candidatos con graphify",
  "definir servicio primario", "identificar APIs impactadas", "identificar eventos impactados",
  "technical-impact-analysis". Do NOT use for Step 1 business resolution, Step 3 technical-story
  enrichment, implementation, or code changes.
argument-hint: "HU/HAB + business-resolution.yaml (+ candidate services)"
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "technical-impact-analysis"
  frameworks: ["Spec-Driven Development", "BMAD", "Graphify"]
---

# bcv-graphify-impact-analysis

## Objective

Validate and refine Step 1 service candidates using Graphify and repository evidence,
producing the minimum viable technical-impact matrix required for Step 3.

This skill implements only **Step 2: Technical Impact Analysis** in the HU/HAB -> HT flow.
It does not perform Step 1 business resolution, Step 3 technical-story authoring, or implementation.

## Mandatory BCV context bootstrap

Before any technical analysis, load context in this strict order:

1. `docs/.agent-context/graph-index.md`
2. `docs/.agent-context/service-map.md`
3. `docs/.agent-context/graphify-query-playbook.md`

Optional standardization context:

4. `docs/cross-service-patterns.md` when available; otherwise use `docs/.agent-context/cross-service-patterns.md`.

Purpose of each source:

- `graph-index.md`: identify relevant services and god nodes quickly.
- `service-map.md`: validate ownership boundaries and cross-service dependencies.
- `graphify-query-playbook.md`: apply focused query templates and budget discipline.
- `cross-service-patterns.md`: standardize naming for adapters, ports, APIs and messaging.

## Input expected

Provide one or more of:

1. Original HU/HAB or functional request.
2. Step 1 output (`business-resolution.yaml`) with candidates and handoff questions.
3. Candidate services to validate.
4. Optional constraints: expected endpoint, event, persistence object, error code.

Minimum required input to run without blocking:

1. HU/HAB text or concise functional summary.
2. At least one candidate service or Step 1 candidate section.

If these minimums are missing, return `BLOCKED` with targeted questions.

## Output expected

Default output artifact name: `technical-impact-analysis.yaml`.
Reference template: `assets/technical-impact-analysis.template.yaml`.

Required top-level keys:

- `primary_service`
- `supporting_services`
- `impacted_apis`
- `impacted_persistence`
- `impacted_events`
- `risks`
- `assumptions`
- `technical_status`

Supporting keys recommended for traceability:

- `evidence_freshness`
- `analysis_scope`
- `conflicts`
- `verification_notes`

## Workflow

### SDD

1. **Specify**: parse Step 1 handoff + HU into technical questions.
2. **Validate**: verify candidate services against context and graph freshness.
3. **Generate**: produce confirmed/candidate impacts by API, persistence and events.
4. **Verify**: apply gate criteria and return `technical_status`.

### Technical-impact execution steps

1. Verify evidence freshness:
   - graph generation timestamp;
   - commit/version when available;
   - draft/snapshot/needs-review warnings.
2. Analyze candidate services:
   - relevant controllers/subscribers/use cases;
   - inbound and outbound integrations;
   - events published/consumed;
   - persistence likely impacted.
3. Assign technical roles:
   - probable technical owner;
   - probable data owner;
   - supporting participants/downstreams.
4. Detect conflicts:
   - ambiguous ownership;
   - contradictory evidence;
   - external dependencies not visible in workspace.
5. Emit gate:
   - `READY`, `REVIEW_REQUIRED`, or `BLOCKED`.

### Graphify query discipline

Use focused, single-objective queries only.

- Budget baseline: `--budget 300`.
- Use `--budget 500` for two architecture layers.
- Reserve `--budget 800` for cross-service transversal cases.

Prioritize this escalation order:

1. If owner can be resolved from index+map, avoid broad graph queries.
2. Read only candidate service `graphify-out/GRAPH_REPORT.md` sections: `Summary` and `God Nodes`.
3. Query by one objective at a time: endpoint, event, persistence, or error handling.
4. Use graph path only to confirm one source-target relation.

## Technical status

- `READY`: sufficient confirmed evidence exists to drive Step 3.
- `REVIEW_REQUIRED`: analysis produced actionable candidates but has unresolved ambiguity or conflict.
- `BLOCKED`: no defensible technical impact can be produced from available evidence.

## Rules and guardrails

1. Graphify accelerates analysis but does not replace architecture judgment.
2. Separate `confirmed` vs `candidate` in impacted APIs, persistence and events.
3. Never present absolute conclusions with incomplete evidence.
4. Mark assumptions with `ASSUMED` and risks with `RISK`.
5. Do not generate implementation tasks or Step 3 technical story sections.
6. Do not modify source code during analysis.
7. Do not scan unrelated services when one owner candidate is strongly supported.
8. If output depends mostly on draft or stale artifacts, downgrade status to `REVIEW_REQUIRED`.

## Handoff to Step 3

The next phase (`bcv-technical-story-enricher`) should receive:

- original HU/HAB summary;
- validated `primary_service` and `supporting_services`;
- impact matrix split by confirmed/candidate;
- explicit assumptions and risks;
- unresolved questions requiring architecture review.

## Completion checklist (Definition of Done)

1. All required top-level keys are present.
2. Each impacted section differentiates confirmed vs candidate.
3. Evidence freshness is explicitly reported.
4. Technical status is consistent with conflicts and evidence quality.
5. Risks and assumptions are explicit and actionable.
6. Output is ready for Step 3 without inventing implementation details.

## Activation quick check

Activate this skill when the request asks to:

- validate Step 1 candidates with Graphify and repository evidence;
- identify APIs/persistence/events with probable technical ownership;
- emit `READY`, `REVIEW_REQUIRED` or `BLOCKED` for technical gate.
