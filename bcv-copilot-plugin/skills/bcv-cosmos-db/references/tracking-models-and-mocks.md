# Tracking Models and Mock-First Strategy for Cosmos DB

## Purpose

This document helps the `bcv-cosmos-db` skill decide:

- how to model tracking/workflow documents
- when to use event log, snapshot, or hybrid models
- how to apply **mock-first** when Cosmos is unavailable
- which artifacts to produce for implementation and review

---

## BCV Rule for This Skill

The primary target is not generic document persistence, but **operational tracking and workflow persistence**.

That means the skill should favor models aligned with:

- tracking identifiers
- workflow status transitions
- registration/update timestamps
- related business identifiers
- minimum audit metadata
- troubleshooting and traceability needs

---

## Required Modeling Questions

Before proposing the final document model, the skill must clarify:

1. Is full change history required?
2. Is **current state** more important than **historical trace**?
3. Are reads driven by expedient, correlation, customer, or another key?
4. Is time-ordered tracking retrieval required?
5. Is retention short-lived or audit-heavy?
6. Is tracking consumed only by the service or also by support/operations?

---

## Option 1 - Event Log

## Description

Each relevant workflow change is stored as a separate event document.

### Best when

- detailed traceability is required
- audit/reconstruction is critical
- workflow has many transitions that must be preserved

### Conceptual fields

- `id`
- `trackingId`
- `correlationId`
- `businessId`
- `workflowStatus`
- `eventType`
- `registeredAt`
- `payload`
- `createdBy`
- `schemaVersion`

### Advantages

- full history
- strong auditability
- flexible sequence reconstruction

### Risks

- more reads to obtain current state
- higher cost for frequent “latest status” reads
- higher aggregation complexity

---

## Option 2 - Updatable Snapshot

## Description

One document represents current tracking state and is updated on each transition.

### Best when

- current-state reads dominate
- read speed is a priority
- fine-grained history is not mandatory

### Conceptual fields

- `id`
- `trackingId`
- `correlationId`
- `businessId`
- `currentStatus`
- `registeredAt`
- `lastUpdatedAt`
- `attributes`
- `schemaVersion`

### Advantages

- simple and fast reads
- less overhead for current-state queries
- fewer documents overall

### Risks

- historical detail can be lost if not complemented
- weaker auditability
- overwrite risks without logical concurrency control

---

## Option 3 - Hybrid

## Description

Store both:

- a snapshot document for current state
- event documents for relevant history

### Best when

- business needs current state and traceability
- support/operations need sequence analysis
- tracking has both operational and forensic value

### Advantages

- balance between fast reads and rich history
- stronger troubleshooting support
- good fit for complex tracking scenarios

### Risks

- higher consistency complexity
- more design decisions
- controlled data duplication

### General skill recommendation

For BCV tracking, **hybrid** is often a strong candidate, but the skill must justify it from explicit access patterns.

---

## Recommended Minimum Audit Metadata

If there is no explicit service standard, the skill should suggest conceptual fields such as:

- `createdAt`
- `updatedAt`
- `createdBy`
- `updatedBy`
- `schemaVersion`
- `sourceSystem`
- `traceId` or `correlationId`

The skill should not force exact names when service conventions already exist.

---

## Document Versioning

The skill should recommend versioning when:

- the document may evolve
- multiple consumers exist
- release-to-release structure changes are likely

### Conceptual guidance

- include `schemaVersion`
- avoid destructive schema changes without compatibility strategy
- document required vs optional fields

---

## Mock-First Strategy

If Cosmos is unavailable or credentials/network access are missing:

1. define a persistence interface/port
2. create a fake or in-memory adapter
3. validate behavioral contracts
4. then prepare real Cosmos integration

This follows the core rule:

> external dependencies must be mocked first

---

## What the Skill Should Produce in Mock Mode

When the user requests implementation but Cosmos is unavailable, the skill should generate:

- repository/adapter interface
- fake/in-memory implementation
- tracking document fixtures
- simple contract tests
- migration guidance to swap fake with real adapter

---

## Fake/In-Memory Rules

The fake should:

- mimic key real-adapter behaviors
- support tracking identifier lookups
- support status filters when required by the use case
- preserve temporal ordering when business logic depends on it
- avoid invented behaviors outside the real contract

---

## Validation Before Switching to Real Cosmos

- document model confirmed
- priority queries confirmed
- partition key decision documented
- TTL decision documented
- retry/timeout policy defined
- 429 handling and availability strategy defined

---

## Example Decision Output

### Scenario

“Need to persist H2H workflow tracking and query by correlation, status, and date.”

### Expected skill response

- identify whether correlation-centric or expedient-centric access dominates
- choose event, snapshot, or hybrid model with justification
- warn about potential cost of global status queries
- generate fake adapter first when Cosmos is unavailable
- then propose real adapter using reactive Spring Data Cosmos or direct SDK
