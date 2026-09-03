---
name: bcv-clean-architecture
description: |
  Use this skill to audit, review or refactor an existing BCV Java/Spring Boot service toward
  clean/hexagonal architecture. Enforces layer dependency rules (core/input/output/app), detects
  violations, and produces a migration plan for legacy or flat-structured services.
  Triggers: "auditar arquitectura", "revisar dependencias entre capas", "migrar a hexagonal",
  "refactor a clean architecture", "core depende de output", "romper dependencia circular",
  "service-point migration", "estructura plana a hexagonal", "dependency rule violation",
  "arquitectura limpia BCV".
  Do NOT use for generating a single new use case slice (use bcv-hexagonal-architecture), for
  OpenAPI contract design (use bcv-openapi-design), or for messaging-only changes.
argument-hint: "service path + goal (audit / refactor / migrate) + target modules"
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "architecture"
  frameworks: ["Spec-Driven Development", "BMAD"]
---

# bcv-clean-architecture

## Language handling and output policy

See [references/language-policy.md](references/language-policy.md).

## Objective

Audit, review and refactor BCV services so they comply with clean/hexagonal architecture:
correct module split (`-core`, `-input`, `-output`, `-app`), ports-and-adapters boundaries and
layer dependency rules. For legacy flat services (e.g. `bcv-bacc-service-point-service`), produce
a safe, incremental migration plan.

This skill is the **review + refactor** counterpart of `bcv-hexagonal-architecture`, which
generates one new vertical slice for a new use case.

## Scope

- Architecture audit: classify a service as hexagonal, layered or flat.
- Dependency rule detection: find `core -> input`, `core -> output`, `input -> output` violations.
- Ports-and-adapters boundary review: detect domain leaking infrastructure concerns.
- Migration planning: flat/legacy → hexagonal module split.
- Package restructuring and module `pom.xml` wiring.
- Preserve behavior: refactors must not change business logic.

## Expected input

- Service repository path (or workspace path).
- Goal: `audit`, `refactor` (targeted fix) or `migrate` (full restructure).
- Optional: `GRAPH_REPORT.md` or a target bounded context/package to focus on.

## Expected output

1. Architecture classification and a violation report.
2. A migration/refactor plan ordered by risk (dependency rule fixes first, module split last).
3. For `refactor`: minimal diffs moving/adapting classes and fixing imports.
4. For `migrate`: new module structure, `pom.xml` changes and a per-package move map.
5. A verification checklist (build + tests + rule checks).

## Workflow

### 1. Audit

Classify the current shape:

| Shape       | Signals                                                                                            | Typical BACC example                                  |
| ----------- | -------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| Hexagonal   | `-core/-input/-output/-app` modules, `port.in`/`port.out`, `UseCaseConfig`                         | reporting, PLM, compliance, current-account, customer |
| Layered     | `controller/service/repository` packages, no ports                                                 | (none of the 6 hexagonal services)                    |
| Flat legacy | single module, `controller/facade/service/repository` packages, `@Service`/`@Component` everywhere | `service-point-service`                               |

Record:

- Base package and module list.
- Whether ports (`port.in`, `port.out`) exist.
- Whether domain classes import Spring/Web/JPA annotations (infrastructure leak).
- The dependency rule violations found.

### 2. Specify (SDD)

Transform the audit into a compact refactor spec:

- Target architecture (clean/hexagonal).
- Affected packages/classes.
- Ordering: (1) fix dependency direction, (2) extract ports, (3) move adapters, (4) split modules.
- Compatibility and build constraints.

### 3. Refactor / Migrate (BMAD)

1. **Understand** — confirm scope and the classes involved.
2. **Design** — decide the target package/module for each class.
3. **Build** — apply the smallest move that reduces violations without changing behavior.
4. **Validate** — rerun the dependency-rule check and the build/tests.

## Layer dependency rules (mandatory)

Allowed:

- `input -> core`
- `output -> core`
- `app -> all` (wiring only)

Forbidden:

- `core -> input`
- `core -> output`
- `input -> output`

Additional clean-architecture rules:

1. Core use cases are plain Java classes (no `@Service`, no `@Component`, no Spring imports).
2. Core domain has zero infrastructure dependencies (no JPA, no Spring Web, no Feign).
3. Services depend on ports/interfaces, never on concrete adapters.
4. DTOs in `input` are Java records (no Lombok on DTOs).
5. Persistence adapters live in `output` and implement `port.out`.
6. Wiring lives in `app` (`UseCaseConfig` or equivalent), never in core/input/output.

See `references/dependency-rules.md`.

## Migration order (flat → hexagonal)

1. Identify domain entities and use cases in the flat service.
2. Create `-core` module: move domain + use cases + `port.in`/`port.out`.
3. Create `-input` module: move controllers/subscribers + DTOs + mappers.
4. Create `-output` module: move repositories/adapters/publishers/clients.
5. Create `-app` module: launcher + config + wiring.
6. Replace direct `@Service`/`@Component` calls with port interfaces.
7. Verify each step builds and tests pass before the next.

See `references/migration-checklist.md`.

## Clarification questions (ask at most 3)

1. What is the goal: `audit`, `refactor` a specific violation, or `migrate` the whole service?
2. Which bounded context or package should I focus on (if not the whole service)?
3. Should the refactor preserve the current public API and message contracts (no breaking changes)?

## Evaluation checklist

- [ ] Service shape correctly classified (hexagonal / layered / flat).
- [ ] Every `core -> input`, `core -> output`, `input -> output` violation is reported.
- [ ] Infrastructure leaks into core are flagged.
- [ ] Refactor plan is ordered by risk (dependency direction first).
- [ ] No business logic changed during refactor.
- [ ] Target code respects the layer dependency rules.
- [ ] `pom.xml` module wiring is correct for migrated services.
- [ ] Build and tests are verified after the change.

## Reference loading policy

Load only what is needed:

1. `references/dependency-rules.md` — layer rules + violation examples + enforcement.
2. `references/migration-checklist.md` — flat → hexagonal step-by-step + pom wiring.

Prefer detected repository conventions over generic examples.

## When not to use this skill

1. Project is already hexagonal and the request is to add one new use case (use `bcv-hexagonal-architecture`).
2. Request is OpenAPI contract design (use `bcv-openapi-design`).
3. Request is messaging-only (use `bcv-azure-service-bus`).
4. Request is infrastructure/CI-CD only.
