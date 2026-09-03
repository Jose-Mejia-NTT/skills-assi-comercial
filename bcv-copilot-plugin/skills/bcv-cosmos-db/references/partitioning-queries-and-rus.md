# Partitioning, Queries, and RU/s for Cosmos Tracking

## Purpose

This reference guides design decisions for tracking/workflow persistence in Azure Cosmos DB within BCV, with emphasis on:

- `partitionKey` selection
- core query patterns
- date-based ordering
- RU/s cost control
- reducing unnecessary cross-partition queries

---

## Core Skill Rule

The skill must **not assume** a mandatory `partitionKey` or TTL when the user has not confirmed them.

Instead, it must:

1. ask for main query patterns
2. ask for expected volume
3. ask whether there is a corporate partitioning rule
4. propose options with explicit trade-offs

---

## Typical Access Patterns

Based on BCV context, common patterns include:

1. lookups by tracking identifiers
2. lookups by workflow status
3. ordering by registration date
4. lookups by business identifiers such as:
   - expedient
   - customer
   - correlation
   - operation

The skill must validate which patterns are actually priority in the current use case.

---

## Partition Key Design Checklist

Before recommending a `partitionKey`, answer:

- What is the highest-frequency query?
- Which query becomes most expensive if cross-partition?
- Are reads centered on a stable business key?
- Is there hotspot risk?
- Can data growth become uneven across partitions?
- Is in-partition time ordering required?
- Is multi-tenant support required?
- Are there heavy batch/replay workloads?

---

## Typical Partition Key Candidates

### Option 1 - `/trackingId` or `/correlationId`

### When it can work

- main access path is by unique flow identifier
- operations are point lookups or updates within the same flow
- all events of one process should be grouped together

### Advantages

- excellent point-read behavior
- simpler flow history reconstruction
- fewer cross-partition reads for correlation-based access

### Risks

- weak support for global status queries
- expensive cross-cutting analytics queries
- potential stress when one correlation accumulates too many events

---

### Option 2 - `/expedientId` or main business identifier

### When it can work

- expedient is the natural domain key
- tracking is mostly queried by expedient
- multiple workflow events must be grouped per business entity

### Advantages

- aligns persistence with domain language
- useful for expedient-level auditing
- reduces mental joins between tracking and business data

### Risks

- not always optimal for global status queries
- possible skew if some expedients become too hot

---

### Option 3 - `/workflowStatus`

### When it can work

- rarely as the primary option
- only when most queries are status-grouped and traffic per status is stable

### Key risks

- high hotspot risk
- uneven distribution if a few statuses dominate
- poor fit for per-flow history reconstruction

The skill should be conservative and avoid this option as a default recommendation.

---

### Option 4 - derived composite logical key

Conceptual examples:

- `businessId#period`
- `correlationId#month`
- `tenant#businessId`

### When it can work

- high-volume workloads
- explicit need to control distribution
- clear time-based or multi-tenant access pattern

### Advantages

- potentially better balance
- controlled growth per logical partition
- useful for high-volume tracking

### Risks

- more operational complexity
- higher query/support cognitive load
- requires strict design and naming discipline

---

## Status + Date Queries

A recurring BCV pattern is:

- filter by `workflowStatus`
- filter/order by registration date

### Skill recommendations

- model a consistent time field, for example:
  - `registeredAt`
  - `statusChangedAt`
- distinguish clearly between:
  - document creation time
  - business event time
  - last update time
- avoid global status queries that scan the full container when volume may grow

### Warning signal

If the user asks for queries like “all tracking records in status X ordered by date” over high volume without partition alignment, the skill must explicitly warn about scalability and RU/s impact.

---

## RU/s Strategy

### What the skill should evaluate

For any proposal, evaluate:

- expected read volume
- read/write ratio
- estimated document size
- update frequency
- multi-filter query complexity
- ordering requirements
- tracking history growth
- TTL requirement

---

## Design Recommendations to Reduce RU/s

- prefer point reads when possible
- align `partitionKey` with critical access patterns
- avoid unnecessary cross-partition queries
- keep documents reasonably sized
- avoid non-value duplicate data
- separate operational tracking from analytics when needed
- choose events vs snapshots based on real business access

---

## Events vs Snapshot

The skill should evaluate the best model for the use case:

| Model | Best when | Risks |
| --- | --- | --- |
| **Event log** | full history and audit detail are required | more reads to rebuild current state |
| **Updatable snapshot** | current-state reads are priority | lower historical traceability |
| **Hybrid** | both current state and constrained history are required | higher consistency complexity |

For BCV tracking scenarios, **hybrid** is often a strong candidate when both operational state and traceability are needed.

---

## TTL and Retention

If not confirmed, the skill must ask:

- Does tracking expire?
- Is retention measured in days, months, or years?
- Are there regulatory requirements?

### Recommendation

- do not enable TTL by default
- explain retention/cost impact of TTL
- TTL can fit purely operational tracking
- validate before proposing expiration in regulated/audit-heavy contexts

---

## Signals to Escalate Design Risk

The skill should raise warnings when it detects:

- unknown volume with heavy global queries
- frequent status queries across many partitions
- broad ordering requirements without clear strategy
- mixed operational tracking and analytical reporting in one model
- hotspot risk from overly concentrated keys

---

## Recommended Decision Output Format

When proposing a `partitionKey`, use a table like this:

| Option | Supports | Risks | Verdict |
| --- | --- | --- | --- |
| `/correlationId` | per-flow history | expensive global status queries | recommended when flow-centric access dominates |
| `/expedientId` | expedient-centric access | skew from hot entities | recommended when expedient is the dominant key |
| `/workflowStatus` | direct status filtering | hotspots and poor distribution | not recommended as default |
