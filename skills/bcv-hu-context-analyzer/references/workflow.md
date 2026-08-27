# Workflow

## 0. Optimization bootstrap (run once per repo or per change)

Before analyzing HUs, ensure the following artifacts exist. They dramatically reduce token usage.

### 0.1 Pre-build graphify graphs

Run for each repository in the workspace:

```bash
graphify <repo-path> --code-only
```

Recommended: run this in CI on every merge to `main` so graphs stay fresh.

### 0.2 Build minimal workspace context

Create a single file manually or via a lightweight script. Do **not** use LLM radiography.

Required file:

- `docs/.agent-context/service-map.md` — role, stack, and god nodes per service in one table.

Keep it under 100 lines.

Optional file per service:

- `<repo>/docs/.agent-context/gotchas.md` — non-obvious constraints, max 10 lines.

See [optimizations.md](optimizations.md) for templates, Git versioning rules, and graphify update triggers.

### 0.3 Version context in Git

Commit stable, lightweight context files:

- `docs/.agent-context/service-map.md`
- `<repo>/docs/.agent-context/gotchas.md` (only if meaningful)

Do not commit `graphify-out/cache/` or large auto-generated files.

---

## 1. Validate the HU

- Clear title.
- Actor, action, and goal.
- At least one acceptance criterion.
- Business rules or scope.

If anything is missing, **stop** and return the validation errors. Do not continue.

### Backend-only scope check

If the HU mentions channels, screens, UI, or frontend flows (e.g., "canal BCW", "pantalla", "bandeja", "dropdown"), do **not** generate frontend ownership gaps.

Instead:

- Identify which backend service provides data to that channel.
- Look for APIs, events, or DTOs consumed by the channel.
- If no backend contract exists, record a technical gap about the missing backend contract, not the missing screen.

---

## 2. Classify services

For each repository in the workspace:

- Read `docs/.agent-context/service-map.md` (max 100 lines).
- Fall back to the `README.md` summary/scope section (max 30 lines) only if `.agent-context/` is missing.
- Classify the service **before** running any graphify query:

| Type | Definition | graphify queries allowed |
| --- | --- | --- |
| `primary` | Owns the business logic or persistence of the change. | Max 2 |
| `secondary` | Only sends/receives an event or acts as a gateway. | Max 1 |
| `omitted` | Transitive dependency with no own logic. | 0 |
| `to_confirm` | Potential impact but unclear from README. | 0 until clarified |

---

## 3. Ensure graphify-out exists

For each primary or secondary service:

```bash
graphify <repo-path> --code-only
```

Skip if `graphify-out/graph.json` already exists and is recent.

For cross-repo analysis (optional):

```bash
graphify merge-graphs \
  <repo1>/graphify-out/graph.json \
  <repo2>/graphify-out/graph.json \
  --out .context/hu-<code>-graph.json
```

---

## 4. Load service gotchas (optional)

If `<repo>/docs/.agent-context/gotchas.md` exists, read it. It contains non-obvious constraints that may affect the analysis.

Do not expect this file to exist for every service.

---

## 5. Query with graphify

Default command priority (all with `--max 5`):

```bash
# Does the main concept already exist in code?
graphify query "<main-concept>" --max 5 --graph <repo>/graphify-out/graph.json

# Path between two key concepts
graphify path "<A>" "<B>" --max 5 --graph <repo>/graphify-out/graph.json

# Explanation of a key node
graphify explain "<GodNode>" --max 5 --graph <repo>/graphify-out/graph.json

# Call relationships
graphify query "<concept>" --max 5 --context_filter call --graph <repo>/graphify-out/graph.json
```

Rules:

- If `query` returns no matches, declare the concept as **new**.
- If the concept exists, identify the injection point (use case, mapper, entity, publisher).
- Do not run more than 2 queries per primary service.
- Do not run more than 1 query per secondary service.

---

## 6. Confirm with targeted reads (max 3)

Use `grep` to locate files and `read offset/limit` to read only the relevant slice.

Before each read, verify:

- Is it essential for the context?
- Is it not already covered by graphify output or gotchas?
- Is the `offset/limit` range minimal?

Track explicitly: **Reads used: N/3**.

---

## 7. Detect gaps

Record a gap when any of the following occurs:

- HU concept not found in any repository.
- Main injection point cannot be identified.
- Missing catalog, master service, or external reference.
- Unclear whether a secondary service is affected.
- Ambiguity about field location (request vs domain vs entity).
- Business rule without clear technical implication.
- Functional requirement that cannot be mapped to an existing service.

### Gap format

Each gap must include:

```markdown
- **ID:** GAP-{NN}
- **Type:** business | technical | implementation
- **Blocking:** yes | no
- **Description:** {concise description}
- **Suggested answer (optional):** {proposed resolution}
- **Affected section:** {which part of the DHU is blocked}
```

### Important

Do **not** ask the user to resolve gaps during this skill. All gaps are forwarded to `bcv-dhu-writer`, which will present them at the end with suggested answers.

---

## 8. Generate the context file

Write `.context/hu-<code>.md` following the structure in [template.md](template.md).

Include a dedicated `Gaps` section with all detected gaps, ordered by blocking first.

If a new non-obvious gotcha was discovered, consider updating `<repo>/docs/.agent-context/gotchas.md`.

---

## 9. Respond to the user

Return a concise summary with:

- Path to the generated context file.
- Classified services (primary/secondary/omitted/to_confirm).
- Main injection point.
- Top 3 gaps (blocking first).
- Whether `service-map.md` or `gotchas.md` were used.
- Note that `bcv-dhu-writer` will present suggested answers for all gaps.
