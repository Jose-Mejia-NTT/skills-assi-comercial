---
name: bcv-technical-story-enricher
description: |
  Use this skill when the user needs Step 3 of the BCV HU/HAB flow: consolidate
  business-resolution and technical-impact-analysis into an implementable technical story.
  Triggers: "generar HT enriquecida", "technical story enriched", "consolidar paso 3",
  "armar historia tecnica", "technical-story-enriched". Do NOT use for Step 1 business
  resolution, Step 2 Graphify impact analysis, direct implementation, or source code changes.
argument-hint: "HU/HAB + business-resolution.yaml + technical-impact-analysis.yaml"
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "technical-story-generation"
  frameworks: ["Spec-Driven Development", "BMAD"]
---

# bcv-technical-story-enricher

## Objective

Generate a single, implementation-ready technical story from validated functional
and technical evidence, preserving full traceability from HU/HAB to tasks.

This skill implements only **Step 3: Technical Story Enrichment** in the HU/HAB -> HT flow.
It does not execute Step 1, Step 2, or implementation.

## Mandatory BCV context bootstrap

Before consolidation, load context in this strict order:

1. `docs/.agent-context/graph-index.md`
2. `docs/.agent-context/service-map.md`
3. `docs/.agent-context/graphify-query-playbook.md`

Optional standardization context:

4. `docs/cross-service-patterns.md` when available; otherwise use `docs/.agent-context/cross-service-patterns.md`.

Purpose:

- enforce shared terminology across services;
- preserve consistency with Step 1 and Step 2 outputs;
- avoid naming drift in APIs, persistence, events and integrations.

## Input expected

Provide one or more of:

1. Original HU/HAB or functional request summary.
2. `business-resolution.yaml` from Step 1.
3. `technical-impact-analysis.yaml` from Step 2.
4. Optional architecture review notes if Step 2 status is `REVIEW_REQUIRED`.

Minimum required input to run without blocking:

1. Functional context (HU/HAB or summary).
2. Step 1 output with candidates and ambiguities.
3. Step 2 output with technical impact matrix and status.

If any minimum input is missing, return `BLOCKED` and request exact missing artifacts.

## Output expected

Default output artifact name: `technical-story-enriched.md`.
Reference template: `assets/technical-story-enriched.template.md`.

Minimum required sections:

1. Functional context summary.
2. Decision on impacted services.
3. Technical impact matrix.
4. Technical task plan.
5. Risks and assumptions.
6. Validation checklist.

Mandatory traceability:

- Every major task must map back to HU rule/criteria and impacted component.
- Distinguish confirmed facts from hypotheses in every impact section.

## Workflow

### SDD

1. **Specify**: ingest HU/HAB + Step 1 + Step 2 outputs.
2. **Validate**: verify input status, conflicts, assumptions and evidence freshness.
3. **Generate**: synthesize the enriched technical story and task plan.
4. **Verify**: run completion checklist and publish artifact.

### Technical-story execution steps

1. Consolidate technical scope:
   - primary service;
   - supporting services;
   - explicit change boundaries.
2. Document impacts:
   - APIs;
   - persistence;
   - events;
   - external integrations.
3. Define technical task plan:
   - contract;
   - domain/application;
   - persistence/messaging;
   - observability/security;
   - testing.
4. Register assumptions and risks:
   - each assumption prefixed as `ASSUMED`;
   - each risk prefixed as `RISK`.
5. Define technical acceptance:
   - rule coverage;
   - non-regression expectations;
   - HU -> HT -> tasks traceability.

### Decision branching

1. If Step 2 `technical_status` is `READY`:
   - generate full technical-story-enriched output.
2. If Step 2 `technical_status` is `REVIEW_REQUIRED`:
   - generate story with review gates explicitly marked;
   - include unresolved items and pending validations.
3. If Step 2 `technical_status` is `BLOCKED`:
   - do not fabricate technical story;
   - return `BLOCKED` with missing technical prerequisites.

## Rules and guardrails

1. Maintain strict traceability from HU/HAB and acceptance criteria.
2. Separate `confirmed` facts from `candidate` hypotheses; never hide uncertainty.
3. Do not invent APIs, tables, topics, classes, contracts or external dependencies.
4. Do not overwrite unresolved conflicts from Step 2; carry them forward explicitly.
5. Do not write implementation code or modify source files.
6. Keep the output concise and directly usable for refinement/planning.

## Output quality bar

The output is valid when:

1. all required sections exist;
2. impacted services decision is justified with evidence references;
3. task plan covers contract, domain/application, persistence/messaging, observability/security and testing;
4. risks and assumptions are explicit and actionable;
5. checklist includes coverage, non-regression and traceability checks;
6. no section claims certainty beyond Step 2 evidence quality.

## Completion checklist (Definition of Done)

1. `technical-story-enriched.md` generated with required sections.
2. Service decision aligns with Step 2 status and conflicts.
3. Every task has a traceability pointer to HU criteria and technical impact.
4. ASSUMED and RISK markers are present where applicable.
5. Validation checklist is concrete and testable.

## Activation quick check

Activate this skill when the request asks to:

- consolidate Step 1 + Step 2 into an implementable HT;
- produce a technical plan with explicit impact and acceptance criteria;
- prepare a refinement-ready artifact with minimal ambiguity.
