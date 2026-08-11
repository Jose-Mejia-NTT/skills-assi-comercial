---
name: bcv-cosmos-db
description: |
  Use this skill when the user mentions Azure Cosmos DB, Cosmos SDK, Spring Data Cosmos reactive,
  tracking persistence, RU/s, partition key design, TTL, Cosmos containers, Cosmos queries, 429 retries,
  or BCV Java/Spring Boot Cosmos integration.

  Keywords: cosmos db, azure cosmos, cosmos sdk, spring data cosmos, reactive cosmos, bcv-commons-cosmos,
  partition key, ru/s, request units, container, ttl, tracking, workflow status, correlation id,
  status history, cross-partition query, rate limiting.
argument-hint: "Cosmos goal + stack + tracking pattern + constraints"
metadata:
  version: "1.1.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "backend"
  frameworks: ["Spec-Driven Development", "BMAD"]
---

# bcv-cosmos-db

## Language rules

- Internal processing, generated code and structural reasoning: English.
- Response to the user: the language of the user's initial message (default: Spanish).
- Preserve BCV package names, library names and document terms as-is.

## Objective

Design, implement, review and troubleshoot Azure Cosmos DB usage in BCV Java/Spring Boot services,
with emphasis on tracking/workflow persistence, RU/s control, partitioning and document modeling.

## Scope

- Spring Data Cosmos reactive flows when the service already uses that stack.
- Azure Cosmos Java SDK direct access when the service needs explicit client control.
- Internal wrapper usage through `bcv-commons-cosmos` when available.
- Tracking/event history, status queries, idempotent upserts and read models.
- Mock-first adapters when Cosmos is unavailable or access is blocked.

## Input expected

The user may provide any of the following:

1. A user story or business need for Cosmos persistence.
2. Existing service context: stack, module layout, hexagonal ports/adapters, current queries.
3. Constraints: volume, latency, retention, compliance, multi-tenant rules, RU budget.
4. Existing code: entities, repositories, adapters, configuration or tests.

## Output expected

Depending on the request, return one or more of these outputs:

1. `Spec: Cosmos Tracking` with containers, document shape, queries and constraints.
2. Design with partition key candidates, versioning, idempotency and RU/s guidance.
3. Build-ready snippets for Spring Data Cosmos reactive or Cosmos SDK direct.
4. Mock-first implementation when Cosmos is unavailable.
5. Validation checklist covering partitioning, RU/s, 429 handling, retries and tests.

## Workflow

### 1) Understand

- Summarize the request in 5-10 bullets.
- Identify tracking use cases, queries, retention, compliance and observability constraints.
- Detect the active stack: Spring Data Cosmos reactive, SDK direct or mixed.
- Confirm whether `bcv-commons-cosmos` should be used.

### 2) Specify

Produce or validate a minimal spec with:

- Container names.
- Document model and required fields.
- `id` and business identifiers.
- Query patterns and ordering.
- Partitioning options and risks.
- RU/s assumptions and measurement approach.
- TTL or retention rules.
- Consistency level, if relevant.
- Error and retry expectations.

Return the spec in a Markdown block named `Spec: Cosmos Tracking`.

### 3) Design

- Recommend the best document shape for the access patterns.
- Evaluate partition key candidates against the main queries.
- Prefer fields that support `status + date` reads without unnecessary cross-partition scans.
- Add versioning, idempotency and audit metadata when relevant.

### 4) Build

Generate the smallest useful implementation for the confirmed stack:

- Spring Data Cosmos reactive + `bcv-commons-cosmos` when that pattern exists.
- Cosmos SDK direct when client control and explicit policies are needed.
- Hexagonal adapter/port when the service uses that architecture.
- Fake or in-memory adapter when Cosmos is not available.

### 5) Validate

- Check that partitioning matches the real queries.
- Minimize cross-partition queries.
- Validate RU/s estimates and read/write paths.
- Address 429 handling with bounded retry and backoff.
- Verify timeouts, idempotency and test coverage.

## Guardrails

1. Do not assume partition key or TTL unless the user confirms them.
2. Treat `bcv-commons-cosmos` as a capability-based internal abstraction, not a hardcoded repo reference.
3. Do not print keys, connection strings or secrets.
4. Prefer mocks first when the real Cosmos account, permissions or network are unavailable.
5. Use concise checklists and tables instead of long prose.

## BCV rules

### `bcv-commons-cosmos`

- Treat it as the common BCV abstraction for client creation, retries, timeouts, logging and mapping helpers.
- Keep the skill generic: describe capabilities, not repository paths.

### Performance and RU/s

- Prefer lookups by `id` or a stable business key.
- Optimize status/date queries for the document shape and partitioning strategy.
- If volume is unknown, ask for it or propose low/medium/high scenarios.

### Security and compliance

- Apply data masking when sensitive fields appear.
- Coordinate with observability conventions when trace or audit metadata is needed.

## Reference loading map

Load only the reference needed for the current request:

| Topic | Reference |
| --- | --- |
| Stack selection and integration patterns | `./references/cosmos-sdk-and-stacks.md` |
| Partitioning, queries and RU guidance | `./references/partitioning-queries-and-rus.md` |
| Tracking models, mocks and testing | `./references/tracking-models-and-mocks.md` |

## Evaluation

The output is valid when it:

1. Produces or validates a structured spec.
2. Does not assume partition key or TTL without confirmation.
3. Supports at least one stack: Spring Data Cosmos reactive or Cosmos SDK direct.
4. Includes RU/s, 429, retries and timeout validation.
5. Uses mocks first when Cosmos is unavailable.

## Activation quick check

Activate this skill when the user mentions at least one of:

- Cosmos DB, Cosmos SDK, RU/s, partition key, container, TTL.
- Tracking, workflow, status history, correlation id.
- Spring Data Cosmos reactive.
- `bcv-commons-cosmos`.
- Cross-partition queries or 429 throttling.
