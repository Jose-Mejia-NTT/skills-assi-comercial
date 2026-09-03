# Cosmos Stacks Supported in BCV

## Purpose

This document describes the two Cosmos integration patterns currently used in BCV services:

1. **Reactive Spring Data Cosmos** with a shared internal wrapper
2. **Direct Azure Cosmos Java SDK** with custom configuration/registry

The `bcv-cosmos-db` skill must support both patterns and choose based on service context, existing stack, and change scope.

---

## Stack A - Reactive Spring Data Cosmos

### When to prefer it

Use this approach when the service:

- is already built with reactive Spring Boot
- already uses `Spring Data Cosmos`
- already has internal wrappers/adapters around common abstractions
- needs fast delivery for CRUD, derived queries, or idiomatic Spring integration

### Advantages

- natural Spring integration
- familiar repository abstractions for Java/Spring teams
- good fit for reactive flows
- less boilerplate in simple scenarios

### Risks

- hiding critical performance details when partitioning and queries are not reviewed
- over-abstracting Cosmos configuration
- treating Cosmos like relational/JPA persistence

### Skill recommendations

- always validate `partitionKey` before confirming the final model
- verify real business query patterns first
- do not assume derived repositories are enough for complex tracking queries
- encapsulate Cosmos access behind ports/adapters for hexagonal/clean architecture services
- use the internal `bcv-commons-cosmos` wrapper when the service already follows that pattern

---

## Stack B - Direct Azure Cosmos Java SDK

### When to prefer it

Use this approach when the service:

- already uses the official SDK directly
- needs fine-grained control over clients, containers, queries, and diagnostics
- requires tuning for performance, retries, timeouts, or RU/s measurement
- has a dedicated persistence layer or central configuration registry

### Advantages

- stronger technical control
- better troubleshooting visibility
- more precise query/options/diagnostics design
- better fit when Cosmos is a critical path dependency

### Risks

- more boilerplate
- higher inconsistency risk across services without a shared standard
- steeper learning curve

### Skill recommendations

- standardize client creation/configuration
- centralize retry, timeout, and 429 handling policies
- unify serialization and document versioning criteria
- always review RU/s cost for key queries
- generate explicit adapters so Cosmos access is not scattered

---

## Internal Wrapper `bcv-commons-cosmos`

The skill should treat `bcv-commons-cosmos` as a **shared persistence/infrastructure abstraction**, without binding to a specific repository.

### Expected capabilities

It may provide one or more of the following:

- Cosmos client creation/configuration
- container access helpers
- repository abstractions
- retry/timeout policies
- logging and telemetry support
- shared serialization
- query helpers
- audit metadata conventions

### Skill rule

- if the service already uses `bcv-commons-cosmos`, the skill must extend that pattern
- if the wrapper is not available in the provided context, the skill may describe integration at a capability level
- never invent concrete wrapper APIs when the user has not provided them

---

## Stack Selection Criteria

The skill should evaluate this matrix before proposing implementation:

| Criterion | Reactive Spring Data Cosmos | Direct SDK |
| --- | --- | --- |
| Service already uses Spring Data Cosmos | Yes | Not required |
| Need low boilerplate | High | Medium |
| Need fine-grained technical control | Medium | High |
| Advanced troubleshooting | Medium | High |
| RU/s tuning and diagnostics | Medium | High |
| Internal wrapper alignment | High | High, if wrapper supports it |

---

## Final Decision Rule

If a stack is already established in the service, **prefer consistency with the existing codebase**.

Only propose stack migration when:

- there is clear technical evidence
- the user explicitly requests migration
- the benefit is greater than migration cost
