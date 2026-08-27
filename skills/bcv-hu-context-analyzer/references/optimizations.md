# Optimizations

These optimizations keep token consumption low when using `bcv-hu-context-analyzer` at scale.

The core principle is: **minimal written context + graphify for everything else.**

---

## 1. Pre-build graphify graphs

### Why

Building a graph from scratch with `graphify <repo>` can be expensive because it scans and indexes the repository. If done once per repo and reused, the skill only pays for cheap `graphify query/path/explain` calls.

### How

Run this for every BCV service:

```bash
graphify <repo-path> --code-only
```

Use `--code-only` to skip doc/paper/image extraction.

### Where to run

- Local bootstrap: one-time command per developer.
- CI pipeline: run on every merge to `main` and store `graphify-out/` as artifact.
- Pre-commit hook: optional for active repos.

### Validation

Check that `graphify-out/graph.json` exists:

```bash
test -f <repo>/graphify-out/graph.json && echo "OK" || echo "Missing graph"
```

---

## 2. Minimal workspace context

### Why

Reading full READMEs wastes tokens. A single `service-map.md` with role, stack, and god nodes is enough to classify services and pick starting points for graphify queries.

### Required file

#### `docs/.agent-context/service-map.md`

```markdown
# Service Map

| Service | Role | Stack | God Nodes |
|---|---|---|---|
| party-lifecycle-management-service | Orchestrates expedient and account lifecycle | Java 21, hexagonal, SQL Server, ASB | CreateBusinessAccountService, SplValidationMapper, ExpedientEntity |
| channel-activity-service | Channel integration bridge (SPL, IFX, notifications) | Java 21, stateless, ASB | PowersSplValidationInputPort, PowersValidationMessageInDto |
| service-point-service | Service point and service tray management | Java 17, layered, SQL Server | ServicePointDto |
| account-opening-reporting-service | Reporting and Teradata ingestion | Java 21, SQL Server + Cosmos | ReportTeradataLEDomain, ReportMapper |
```

Keep this file **under 100 lines**.

### How to create

- Do **not** use an LLM to radiograph the repo.
- Extract the table manually from READMEs or existing documentation.
- Update only when service boundaries or god nodes change.

---

## 3. Optional service-level gotchas

### Why

Some constraints are not visible in the code or service name. A very short `gotchas.md` per service captures these non-obvious rules without creating a full baseline document.

### File

```text
<repo>/docs/.agent-context/gotchas.md
```

### Example

```markdown
# party-lifecycle-management-service gotchas

- registryOffice is computed from legalEntityDomain.address.province.
- SPL validation always flows through channel-activity-service.
- @StandardApi requires RFC 9457 error envelopes on command controllers.
```

### Rules

- **Maximum 10 lines.**
- Only include non-obvious constraints that affect implementation decisions.
- Do not create the file if there are no meaningful gotchas.
- Do not use an LLM to generate it.

### When to update

- A new gotcha is discovered during a HU.
- A constraint changes or becomes invalid.

---

## 4. Use `--code-only` graphify builds

### Why

`graphify` can extract docs, papers and images with LLM assistance. For code analysis, this is unnecessary and expensive.

### How

Always build graphs with:

```bash
graphify <repo-path> --code-only
```

This indexes only source code using local AST parsing.

### Exception

If the HU depends on a non-code specification stored in the repo (e.g., `api-contract.md`), extract that separately with a single `read` call, not via graphify build.

---

## 5. Version context in Git

### What to commit

Commit stable, lightweight context files:

```bash
git add docs/.agent-context/service-map.md
git add bcv-bacc-*/docs/.agent-context/gotchas.md
git add .context/hu-<code>.md
git add hu-technical-refinement/HU-...-refined-...md
```

### What NOT to commit

Do not commit large auto-generated or regenerable files:

```gitignore
# graphify cache is regenerable
graphify-out/cache/
.graphify_analysis.json
.graphify_labels.json
```

> `graphify-out/graph.json` may be committed if the team agrees, but it is large. Prefer generating it in CI and storing it as an artifact.

### Update policy

Update context files only when:

- A new microservice is added.
- A service changes its primary responsibility.
- God nodes change significantly.
- A new non-obvious gotcha appears.

Do **not** update them for every HU or code change.

---

## 6. When to update graphify and which microservice

### Trigger conditions

Rebuild the graph for a microservice when any of the following happens:

| Condition | Microservice to update |
|---|---|
| New REST controller, subscriber, publisher, or use case | The service where the change occurs |
| New database entity or table | The service that owns the entity |
| New Feign client or external integration | The service that consumes it |
| Rename or removal of a god node | The affected service |
| Change in package structure or naming conventions | The affected service |
| Merge to `main` with significant refactoring | All affected services |

### What does NOT require a rebuild

| Situation | Reason |
|---|---|
| Small bugfix inside an existing method | The AST node already exists |
| Adding a field to an existing DTO/entity | The node already exists; queries will reflect the field name |
| New unit tests | No impact on the production code graph |
| Documentation-only changes | No impact on code graph |

### How to decide which microservice

1. Identify the service that owns the changed code.
2. If the change is cross-service (e.g., new queue contract), update both the producer and consumer.
3. If unsure, run `graphify check-update <repo>` to see if the graph is stale.

### Recommended CI policy

```yaml
# Example CI step
- name: Update graphify graphs
  run: |
    for repo in bcv-bacc-*; do
      graphify "$repo" --code-only
    done
  if: github.ref == 'refs/heads/main'
```

---

## Expected token savings

| Optimization | Token reduction |
|---|---|
| Pre-built graphs | High — avoids repeated full-repo indexing |
| Minimal `service-map.md` | Medium — replaces full README reads |
| Optional `gotchas.md` | Low-Medium — captures hidden constraints cheaply |
| `--code-only` | Medium — skips non-code LLM extraction |

Combined, these optimizations make the skill cost roughly **10-20%** of an LLM-based repository radiography approach.
