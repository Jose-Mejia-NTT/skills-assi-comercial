# Ambiguity and conflict handling

## Purpose

Define how `bcv-technical-impact-and-story` must handle ambiguities inherited from
`bcv-business-resolution`, plus technical conflicts discovered during Phase A.

The goal is to produce an honest plan that clearly states what can be implemented,
what is blocked and what information is still missing.

## Sources of uncertainty

1. **Inherited business ambiguities**
   - Come from `business-resolution.yaml` under `ambiguities`.
   - Example: unclear ownership of an entity, undefined escalation mechanism.

2. **Technical conflicts discovered in Phase A**
   - Contradictory evidence between graph artifacts and service maps.
   - Two services could own the same capability.
   - Externally-owned dependency not visible locally.

3. **Missing information**
   - No catalog entry for a mentioned capability.
   - Missing graph evidence for a candidate service.
   - Unknown contract for an event or API.

## Handling rules

1. **Record everything in the YAML**
   - Add `inherited_ambiguities: []` to `technical-impact-analysis.yaml`.
   - Add discovered conflicts to `conflicts: []`.
   - Never hide uncertainty or resolve it silently.

2. **Create a discovery task `IMP-000`**
   - The first task in the technical task plan must be a discovery task.
   - It lists every inherited ambiguity and technical conflict.
   - It specifies the required action, suggested owner and clarification questions.
   - It documents the impact on other tasks if unresolved.
   - It provides **suggested resolution options** so the resolver can decide quickly.

3. **Block dependent technical tasks**
   - Any task that cannot be completed until an ambiguity is resolved must have:
     - `Status: BLOCKED`
     - `Dependencies: IMP-000`
     - `Unblock condition:` explicit statement

4. **Do not invent decisions**
   - If ownership is ambiguous, do not pick one owner arbitrarily.
   - If a contract is unknown, do not invent fields or endpoints.
   - If evidence is stale or draft, downgrade to `REVIEW_REQUIRED`.

5. **Generate scenarios when useful**
   - If exactly two or three alternatives are plausible, you may document them as
     `Scenario A`, `Scenario B` in the detailed action plan.
   - Do not generate scenarios when there is no information at all; use `BLOCKED` instead.

6. **Escalation**
   - Mark `escalation_needed: true` in `verification_notes` when:
     - a business decision is required,
     - an external team owns a dependency,
     - the ambiguity affects the primary service selection.

## IMP-000 template

```markdown
### IMP-000: Resolver ambigüedades y conflictos heredados (Discovery)

- **Service:** arquitectura / equipos candidatos
- **Type:** discovery
- **Description:** Investigar y resolver ambigüedades del business resolution y conflictos técnicos.
- **Inherited ambiguities / conflicts:**
  - `[id]`: [descripción] → [acción requerida] → [owner sugerido]
- **Clarification questions:**
  - [Pregunta 1]
  - [Pregunta 2]
- **Suggested resolution options:**
  - **Option A:** [descripción concreta, ejemplo de decisión o artefacto resultante]
  - **Option B:** [alternativa plausible]
  - **Option C (fallback):** [acción conservadora si no hay consenso, ej. "mantener VO actual y documentar deprecación"]
- **Impact if unresolved:**
  - [Tareas bloqueadas: IMP-XXX, IMP-YYY]
- **Unblock condition:**
  - [Condición clara]
- **HU traceability:**
- **Impact traceability:** inherited_ambiguities + conflicts
- **Dependencies:**
- **Status:** TODO
```

## Blocked task template

```markdown
### IMP-XXX: [Título técnico]

- **Service:**
- **Type:**
- **Description:**
- **HU traceability:**
- **Impact traceability:**
- **Dependencies:** IMP-000
- **Unblock condition:** [Condición específica que desbloquea la tarea]
- **Status:** BLOCKED
```

## When to use BLOCKED vs REVIEW_REQUIRED

- Use `BLOCKED` for the `technical_status` when no defensible impact analysis can be produced.
- Use `REVIEW_REQUIRED` when the analysis can proceed but contains unresolved ambiguities.
- Within the story, use `Status: BLOCKED` for individual tasks that depend on unresolved ambiguities,
  even when the overall gate is `REVIEW_REQUIRED`.
