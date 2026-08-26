# Evaluation criteria

## Quality checks for the enriched technical story

Deterministic checks before declaring the technical story ready for refinement.

### Required checks

1. **Evidence isolation**
   - Every technical item in the story exists in the emitted `technical-impact-analysis.yaml`.
   - No item was added from Phase A working memory without being recorded in the YAML first.
   - The story states the YAML path and its `technical_status`.

2. **Scope coherence**
   - Primary and supporting services in the story match the YAML.
   - Any deviation is explicitly justified as a review item.

3. **Impact integrity**
   - APIs, persistence and events preserve the confirmed vs candidate split.
   - No technical item appears without an upstream evidence reference.
   - Inherited ambiguities and technical conflicts are recorded in the YAML.

4. **Task coverage**
   - The task plan includes numbered, actionable tasks (e.g. `IMP-001`).
   - If `inherited_ambiguities` or `conflicts` exist, the plan starts with `IMP-000` (discovery).
   - Tasks that depend on unresolved ambiguities are marked `BLOCKED` with an explicit unblock condition.
   - Each task has a **recommended skill category** mapped from `references/skill-mapping.md`.
   - If the user provided `available_skills`, the category is replaced by the matching skill name; otherwise the category is kept as a placeholder.
   - Tasks cover contract, domain/application, persistence/messaging, observability/security and testing.
   - Each task links to at least one HU criterion and one impact item.
   - Tasks do not include estimates, assignments or code snippets.

5. **Risk and assumption clarity**
   - Every unresolved dependency is either `ASSUMED` or `RISK`.
   - A `REVIEW_REQUIRED` gate produced explicit pending validation items.

6. **Acceptance quality**
   - The checklist includes coverage, non-regression and traceability checks.
   - Criteria are verifiable, not generic placeholders.

7. **Architecture diagram quality**
   - The Mermaid diagram reflects services, APIs, events and persistence from the impact matrix.
   - Candidate items are visually distinguished (dashed lines or labels).
   - The diagram source is written to `docs/historial/<hu-slug>-technical-architecture-diagram.mmd`.
   - The diagram does not introduce elements absent from the impact analysis.

8. **Detailed action plan quality**
   - Every task in the technical task plan has step-by-step implementation, DoR, DoD and technical acceptance criteria.
   - Steps are high-level actions, not code snippets.
   - Test cases, error scenarios, external dependencies, deployment considerations and files affected are documented.
   - Every `Files affected` entry specifies the operation (`create | modify | delete`) and a concrete change description (what is added, removed or altered in that file).
   - Security and compliance notes are included when PII, audit or sensitive data is involved.
   - The plan does not include estimates, assignments or code snippets.

9. **Repository file impact quality**
   - Files to create, modify and delete are listed with owning service and reason.
   - Candidate files are marked explicitly.
   - Domains/entities affected and migrations/configuration changes are documented.

10. **Open questions / clarifications**
    - Any remaining indication, suspicion or missing clarification is listed in the `Open Questions / Clarifications Needed` section.
    - Blocking questions are reflected in `IMP-000` or `technical_status`.
    - Non-blocking questions have a documented fallback assumption.
    - Open questions are presented **one at a time in the chat response**, waiting for each answer before asking the next.
    - The skill must update the YAML and story after receiving all answers in the same chat.

11. **Developer review section**
    - The story includes the developer review & sign-off checklist.
    - The checklist covers HU coverage, executability, dependencies, BLOCKED tasks, error scenarios,
      technical decisions, file impact and missing configuration/documentation.

### Escalation rules

- Gate `BLOCKED`: do not emit a story.
- Gate `REVIEW_REQUIRED`: emit a draft with the pending-review block at the top and unresolved items listed.
- Evidence stale or contradictory: keep `REVIEW_REQUIRED` and escalate to technical review.

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

## When NOT to activate

Do not activate for:

- business or functional resolution (use `bcv-business-resolution`);
- implementation or source code changes;
- API design in isolation;
- test generation.
