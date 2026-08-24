---
name: bcv-openapi-design
description: |
  Use this skill when the user asks to create, complete, review or standardize a REST API contract
  using OpenAPI for a BCV backend service, especially in Spring Boot projects with missing, empty or
  inconsistent `openapi.yaml`, `swagger.yaml` or `rest-api.yaml` files.

  Triggers: "complete rest-api.yaml", "design OpenAPI", "document controllers", "define DTOs",
  "standardize error responses", "crear contrato REST", "diseñar API REST", "documentar endpoints",
  "generar openapi", "alinear springdoc", "validar contrato openapi".

  Do NOT use this skill for implementing the full backend logic, generating database schemas,
  messaging-only changes, or pure unit/integration test generation.
argument-hint: "API goal + current spec/code state + domain rules + error/versioning constraints"
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "api-design"
  frameworks: ["Spec-Driven Development", "BMAD"]
---

# bcv-openapi-design

## Language rules

- Internal processing, structural reasoning and generated technical artifacts: English.
- Response to the user: the language of the user's initial message (default: Spanish).
- Preserve BCV business terms, package names and existing public API names when they are already established.

## Objective

Design, complete, review and standardize OpenAPI contracts for BCV REST services.
This skill is specialized in contract-first and contract-alignment work: it produces a clear,
implementable and reviewable API contract before or alongside implementation.

## Scope

- Create a new OpenAPI contract from a user story, DHU or functional specification.
- Complete an empty or partial `rest-api.yaml`, `openapi.yaml` or `swagger.yaml`.
- Extract and normalize a contract from existing Spring Boot controllers and DTOs.
- Standardize resource naming, methods, status codes, DTO schemas and error responses.
- Prepare a contract that is implementation-ready for Spring Boot + springdoc environments.
- Produce validation findings when an existing contract is incomplete or inconsistent.

## When this skill is a strong fit

Use this skill when one or more of these conditions are true:

1. The repository has a missing, empty or outdated OpenAPI file.
2. The team needs a contract before implementation or before exposing changes to consumers.
3. Controllers exist but the API is undocumented or inconsistent across endpoints.
4. DTOs, error models or versioning rules need to be standardized.
5. The output is expected to be a YAML or contract review rather than Java business logic.

## When not to use this skill

Do not use this skill when:

1. The user mainly wants backend implementation without API contract work.
2. The task is mostly database modeling, JPA, SQL Server or Cosmos persistence.
3. The task is mainly Azure Service Bus, messaging, queues or topics.
4. The task is mainly hexagonal slice generation or use case implementation.
5. The task is only about generating tests with no contract design/review activity.

## Input expected

The user may provide any combination of:

1. A DHU, user story or business requirement.
2. An incomplete or empty OpenAPI file.
3. Existing Spring Boot controllers, DTOs or endpoint descriptions.
4. Project conventions for versioning, security, error handling or naming.
5. Constraints such as backward compatibility, async behavior, idempotency or consumer expectations.

## Output expected

Depending on the request, return one or more of these outputs:

1. A full `openapi.yaml` / `rest-api.yaml` contract.
2. A partial contract update for the requested endpoints only.
3. A structured contract review report with findings and fixes.
4. A minimal specification summary before generation when requirements are ambiguous.
5. A checklist of implementation alignment points for Spring Boot + springdoc.

## Workflow

### 1) Understand

- Summarize the requested API capability in 5-10 bullets.
- Identify resources, business actions, DTOs, main success responses and business errors.
- Detect whether the task is contract-first, contract-completion or contract-review.
- Detect whether implementation already exists and whether the contract must align to it.

### 2) Specify

Produce or validate a compact functional contract specification with:

- Resource names.
- Endpoints and HTTP methods.
- Main request fields and required constraints.
- Main response fields.
- Error scenarios and expected status codes.
- Versioning expectations.
- Async or idempotency rules, if relevant.
- Security expectations, if relevant.

Return this as a Markdown block named `Spec: OpenAPI Contract` when the request is ambiguous or high-impact.

### 3) Design

- Prefer nouns for resources and consistent collection/item patterns.
- Use action-style endpoints only when a business action cannot be modeled as a normal CRUD update.
- Keep request DTOs minimal and explicit.
- Keep response DTOs stable and consumer-oriented.
- Reuse a shared error schema whenever project conventions allow it.
- Align naming and response codes with existing repository conventions when they are already established.

### 4) Build

Generate the smallest correct contract for the request:

- Full YAML when the contract is missing or empty.
- Partial YAML updates when only specific endpoints are requested.
- Review findings when the task is contract validation rather than generation.
- Examples only when they add implementation or consumer clarity.

### 5) Validate

Check that the output:

- matches the user request and business semantics
- uses coherent resource names and HTTP methods
- defines required request fields explicitly
- documents success and error responses
- does not invent unsupported filters, fields or behavior without stating assumptions
- is implementable in a Spring Boot + springdoc service

## SDD

1. Specify: transform the request or current code/spec state into a structured API contract definition.
2. Validate: identify missing business rules, naming choices and response behaviors.
3. Generate: produce or update the OpenAPI artifact only after the contract is clear enough.
4. Verify: review contract completeness, consistency and implementation readiness.

## BMAD

1. Understand: clarify the API goal, entities, client interactions and business rules.
2. Design: define resources, operations, schemas and errors.
3. Build: write the contract or review output.
4. Validate: confirm consistency with repository conventions and expected consumer usage.

## Guardrails

1. Do not invent business rules if the request is clearly underspecified; state assumptions or ask up to 3 clarifying questions.
2. Do not change established public resource names without justification.
3. Do not use verbs as primary resource names unless the repository already follows that style.
4. Do not omit error responses for high-value business operations.
5. Do not leak internal implementation details into public response schemas unless the contract already exposes them.
6. Prefer concise YAML and concise review notes over long explanations.

## BCV-specific expectations

### Contract-first and mocks-first

- If the repository lacks a reliable API contract, prefer defining the contract before implementation.
- If external dependencies affect endpoint behavior and cannot be validated yet, make the dependency visible in the contract assumptions and keep the contract mock-friendly.

### Spring Boot alignment

- Keep the contract easy to map to controller methods, DTOs and validation annotations.
- Favor schema shapes that can be represented naturally in Java DTOs.
- Align status codes and error responses with likely controller advice / exception handling patterns.

### Backward compatibility

- If the API already exists, preserve existing public paths and response shapes unless the user explicitly requests a breaking change.
- Call out breaking changes clearly.

## Reference loading map

Load only what is needed for the current request:

| Topic | Reference |
| --- | --- |
| REST resource naming, methods, DTOs, versioning and shared error model | `./references/rest-contract-guidelines.md` |

## Evaluation

The output is valid when it:

1. Produces a clear contract or a clear contract review.
2. Uses coherent resource naming and HTTP semantics.
3. Documents request/response schemas and main errors.
4. Aligns to existing repository conventions when known.
5. Avoids unsupported assumptions or states them explicitly.

## Activation quick check

Activate this skill when the user mentions at least one of:

- OpenAPI, Swagger, `rest-api.yaml`, `openapi.yaml`
- document endpoints, design API, standardize contract
- DTOs, response schemas, error responses, API versioning
- springdoc alignment or controller documentation
