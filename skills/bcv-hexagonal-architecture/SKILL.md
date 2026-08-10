---
name: bcv-hexagonal-architecture
description: |
  Use this skill when the user asks to add a new use case, endpoint, or feature
  in a Java service that follows a hexagonal architecture style (core/input/output/app
  or equivalent ports-and-adapters layering).
  Triggers: "add use case", "new endpoint", "new feature", "agregar caso de uso",
  "nuevo endpoint", "nueva funcionalidad", "quiero agregar...", "necesito que el servicio pueda...".
  Do NOT use for OpenAPI-first design, test generation, or messaging-only changes.
applyTo: "**"
---

# bcv-hexagonal-architecture

## Objective

Generate one complete hexagonal vertical slice for one new use case,
following the conventions detected in the current repository.

One invocation = one use case = one full slice.

## Expected input

Natural language feature request or DHU with acceptance criteria and business rules.

## Expected output

A complete vertical slice, including:
- core port/in interface
- core use case service
- core port/out interface when persistence is needed
- input DTOs and mapper updates
- input command/query service updates
- input controller endpoint update
- output persistence adapter update
- app UseCaseConfig bean registration

## Compatibility gate (mandatory)

Before generation, verify the repository has enough hexagonal signals:
1. Core/input/output/app modules or equivalent package layering.
2. Port-based boundaries (port.in and/or port.out style).
3. UseCaseConfig or equivalent explicit wiring point.

If these signals are weak or absent, stop and explain why this skill should not be used.

## Project alignment (mandatory before code generation)

Infer from code, do not hardcode:
1. Base Java package.
2. Exact module names.
3. Current path conventions for ports, adapters, controller DTOs, and mappers.
4. Persistence exception strategy already used by existing adapters.

If confidence is low, ask up to 3 clarifying questions.

## Workflow

### SDD

1. Specify: transform request into structured use case spec.
2. Validate: check missing data and ask up to 3 questions.
3. Generate: create slice in order core -> input -> output -> app.
4. Verify: enforce dependency rules and wiring registration.

### BMAD

1. Understand: action, entity, inputs, outputs, business rules.
2. Design: names, signatures, DTOs, endpoint contract.
3. Build: create or update required files by layer.
4. Validate: architectural consistency and annotation checklist.

## Mandatory implementation rules

1. Use cases are plain Java classes: no @Service, no @Component.
2. Services depend on ports/interfaces, never on concrete adapters.
3. Controller methods must include @Operation, @ApiResponses (200/400/500), and @ObservableOperation.
4. DTOs must be Java records (no Lombok on DTOs).
5. Persistence adapters must follow existing project exception style.
6. Register new use case bean in UseCaseConfig.

## Layer dependency rules

Allowed:
- input -> core
- output -> core
- app -> all (wiring only)

Forbidden:
- core -> input
- core -> output
- input -> output

## Reference loading policy

Load only what is needed for the current request:
1. references/project-alignment-checklist.md
2. references/controller-api-conventions.md
3. references/error-handling-patterns.md
4. references/hexagonal-rules.md

Prefer detected repository conventions over generic examples.

## When not to use this skill

1. Project is not hexagonal-compatible (no clear core/input/output/app or equivalent).
2. Request is mainly OpenAPI design from scratch.
3. Request is mainly automated test generation.
4. Request is mainly Azure Service Bus publisher/subscriber changes.
5. Request is infrastructure-only (CI/CD, IaC, deployment).
6. Request requires deep refactor of existing use cases rather than adding one new slice.

## Language rules

- Internal processing and generated code: English.
- User response language: user's initial language (default Spanish).

## Evaluation checklist

- [ ] Complete slice artifacts generated for one use case.
- [ ] Layer dependency rules respected.
- [ ] UseCaseConfig wiring added.
- [ ] Controller annotations present.
- [ ] DTOs are records.
- [ ] Service is plain Java class (no Spring stereotype).
- [ ] Persistence adapter follows local exception strategy.
- [ ] Domain exceptions and unexpected exceptions are handled consistently with project style.
