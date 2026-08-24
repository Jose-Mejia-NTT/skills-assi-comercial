---
name: bcv-technical-impact-and-story
description: |
  Use this skill for the technical half of the BCV HU/HAB flow: validate business-resolution
  candidates against architecture-graph and repository evidence, emit the technical impact
  matrix, and consolidate it into an implementable technical story.
  Triggers: "analizar impacto tecnico", "definir servicio primario", "identificar APIs impactadas",
  "identificar eventos impactados", "technical-impact-analysis", "generar HT enriquecida",
  "technical story enriched", "armar historia tecnica", "technical-story-enriched".
  Do NOT use for business/functional resolution (use bcv-business-resolution), implementation,
  API design, source code changes, or test generation.
argument-hint: "HU/HAB + business-resolution.yaml (or an existing technical-impact-analysis.yaml)"
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "technical-analysis"
  frameworks: ["Spec-Driven Development", "BMAD", "Graphify"]
---

# bcv-technical-impact-and-story

## Objective

Turn a validated business resolution into two linked artifacts:

1. `technical-impact-analysis.yaml` — evidence-graded technical impact matrix plus a gate status.
2. `technical-story-enriched.md` — implementation-ready technical story derived **only** from that YAML.

This skill covers the technical phase of the HU/HAB -> HT flow. It does not perform business
resolution and does not write implementation code.

## Execution modes

Select the mode deterministically from the inputs present. State the selected mode in the response.

| Mode | Condition | Phases run | Artifacts |
| --- | --- | --- | --- |
| `full` | a `business-resolution.yaml` (or candidate list) is provided and no `technical-impact-analysis.yaml` exists | A then B | both |
| `story-only` | a `technical-impact-analysis.yaml` is provided | B only | `.md` |
| `impact-only` | the user explicitly asks only for impact analysis / gate | A only | `.yaml` |

In `story-only`, never re-run graph discovery. Treat the provided YAML as the sole technical truth,
even if it was hand-edited after architecture review.

## Mandatory BCV context bootstrap

Load context in this strict order before Phase A:

1. `docs/.agent-context/graph-index.md`
2. `docs/.agent-context/service-map.md`
3. `docs/.agent-context/graphify-query-playbook.md`
4. `docs/cross-service-patterns.md` when available; otherwise `docs/.agent-context/cross-service-patterns.md`

Purpose:

- `graph-index.md`: locate relevant services and god nodes quickly.
- `service-map.md`: validate ownership boundaries and cross-service dependencies.
- `graphify-query-playbook.md`: query templates and budget discipline (Phase A only).
- `cross-service-patterns.md`: standardize naming for adapters, ports, APIs and messaging.

In `story-only` mode, load only `service-map.md` and `cross-service-patterns.md`.

## Input expected

1. Original HU/HAB or concise functional summary.
2. `business-resolution.yaml` from `bcv-business-resolution`, or an explicit candidate service list.
3. Optional `technical-impact-analysis.yaml` (switches the run to `story-only`).
4. Optional constraints: expected endpoint, event, persistence object, error code.
5. Optional architecture review notes when a previous gate returned `REVIEW_REQUIRED`.

Minimum to run without blocking:

- `full` / `impact-only`: HU/HAB text plus at least one candidate service.
- `story-only`: HU/HAB summary plus a `technical-impact-analysis.yaml` with a `technical_status`.

If the minimum is missing, return `BLOCKED` with targeted questions and produce no artifact.

## Output location

Write both artifacts to `docs/historial/` using the HU slug as prefix:

- `docs/historial/<hu-slug>-technical-impact-analysis.yaml`
- `docs/historial/<hu-slug>-technical-story-enriched.md`

The `.md` must record the exact path and `technical_status` of the `.yaml` it consumed.

Write each artifact automatically before responding. In `full` mode, write the YAML first and
apply the gate barrier before creating the Markdown. In `impact-only` mode, write only the YAML.
In `story-only` mode, write only the Markdown.

## Workflow

### Phase A — Technical impact analysis

Reference template: `assets/technical-impact-analysis.template.yaml`.
Evidence grading and gate criteria: `references/evidence-and-gates.md`.
Query budgets and escalation order: `references/graphify-query-discipline.md`.

1. Verify evidence freshness: graph timestamp, commit/version, draft/snapshot/needs-review warnings.
2. Analyze candidate services: controllers/subscribers/use cases, inbound and outbound integrations,
   events published/consumed, persistence likely impacted.
3. Assign technical roles: probable technical owner, probable data owner, participants, downstreams.
4. Detect conflicts: ambiguous ownership, contradictory evidence, externally-owned dependencies.
5. Emit the gate: `READY`, `REVIEW_REQUIRED` or `BLOCKED`.

Required top-level keys: `primary_service`, `supporting_services`, `impacted_apis`,
`impacted_persistence`, `impacted_events`, `risks`, `assumptions`, `technical_status`.
Recommended for traceability: `evidence_freshness`, `analysis_scope`, `conflicts`, `verification_notes`.

### Gate barrier (mandatory between phases)

This barrier replaces the human handoff that existed when the two phases were separate skills.
Do not soften it.

1. Write the complete `.yaml` to disk **before** drafting any part of the story.
2. Report `technical_status` to the user explicitly.
3. Branch:
   - `BLOCKED` → stop. Deliver only the YAML and the missing technical prerequisites. Never fabricate a story.
   - `REVIEW_REQUIRED` → continue, but the story must open with a "Pending architecture review" block
     listing every conflict and candidate-only impact. Never bury it at the end.
   - `READY` → continue with the full story.

### Phase B — Technical story enrichment

Reference template: `assets/technical-story-enriched.template.md`.
Completion checks: `references/quality-checks.md`.

1. Consolidate technical scope: primary service, supporting services, explicit change boundaries.
2. Document impacts: APIs, persistence, events, external integrations.
3. Define the technical task plan: contract, domain/application, persistence/messaging,
   observability/security, testing.
4. Register assumptions (`ASSUMED`) and risks (`RISK`).
5. Define technical acceptance: rule coverage, non-regression expectations, HU -> HT -> tasks traceability.

Minimum sections: functional context summary, impacted-services decision, technical impact matrix,
technical task plan, risks and assumptions, validation checklist.

## Rules and guardrails

1. **Evidence isolation.** Phase B writes the story exclusively from the emitted YAML. Anything not
   present in the YAML does not exist for the story. If Phase A discovered something worth keeping,
   go back and add it to the YAML first.
2. Graph evidence accelerates analysis but does not replace architecture judgment.
3. Run graph queries only in Phase A. Phase B performs no discovery: no graph artifacts, no queries,
   no source scanning. Missing evidence becomes an open item, not an improvised answer.
4. Separate `confirmed` vs `candidate` in every impacted dimension, in both artifacts.
5. Never present absolute conclusions with incomplete evidence.
6. Mark assumptions with `ASSUMED` and risks with `RISK`.
7. Do not perform business resolution: never invent capabilities or re-map ownership from scratch.
8. Do not modify source code, and do not generate implementation code.
9. Do not scan unrelated services when one owner candidate is strongly supported.
10. If output depends mostly on draft or stale artifacts, downgrade the gate to `REVIEW_REQUIRED`.
11. Carry Step 1 ambiguities and Phase A conflicts forward; never resolve them silently.

## Language rules

- Internal reasoning and YAML key names: English.
- Narrative YAML values, generated Markdown content and the response to the user: the language of
  the developer's initial message; default to Spanish.
- Preserve status values, artifact paths, BCV names, business terms, service names and public
  identifiers exactly as supplied.
- Do not translate YAML keys or status values (`READY`, `REVIEW_REQUIRED`, `BLOCKED`).
- Preserve BCV names, business terms, service names and public identifiers exactly as supplied.

## Token-efficiency rules

- Resolve ownership from `graph-index.md` + `service-map.md` before opening any graph report.
- Read only the `Summary` and `God Nodes` sections of the candidate service `graphify-out/GRAPH_REPORT.md`.
- One query per objective; never batch unrelated objectives into a broad query.
- Prefer structured fields and short evidence statements over narrative repetition.

## Completion checklist (Definition of Done)

1. Execution mode is stated and consistent with the inputs.
2. `.yaml` contains all required top-level keys and was written before the story.
3. Every impacted section differentiates confirmed vs candidate in both artifacts.
4. Evidence freshness is explicitly reported.
5. `technical_status` is consistent with the conflicts and evidence quality found.
6. The story's service decision matches the YAML, or deviations are flagged as review items.
7. Task plan covers contract, domain/application, persistence/messaging, observability/security, testing,
   and each task traces to an HU criterion and an impact item.
8. `REVIEW_REQUIRED` produced a visible pending-review block; `BLOCKED` produced no story at all.
9. The story references the YAML path and status it consumed.
10. Generated narrative content and the user response use the developer's initial language.

## Activation quick check

Activate this skill when the request asks to:

- validate business-resolution candidates against graph and repository evidence;
- identify impacted APIs, persistence or events with probable technical ownership;
- emit a `READY` / `REVIEW_REQUIRED` / `BLOCKED` technical gate;
- produce an implementation-ready technical story from a validated impact analysis.
