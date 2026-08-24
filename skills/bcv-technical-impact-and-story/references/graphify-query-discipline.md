# Graph query discipline (Phase A only)

## Purpose

Keep technical discovery cheap and defensible. These rules apply only while producing
`technical-impact-analysis.yaml`. Phase B never runs queries.

## Budgets

- Baseline: `--budget 300`.
- Two architecture layers: `--budget 500`.
- Cross-service transversal cases only: `--budget 800`.

## Escalation order

Stop as soon as the current question is answered.

1. Resolve the owner from `docs/.agent-context/graph-index.md` + `docs/.agent-context/service-map.md`.
   If that is conclusive, do not open any graph report.
2. Read only the `Summary` and `God Nodes` sections of the candidate service `graphify-out/GRAPH_REPORT.md`.
3. Query one objective at a time: endpoint, event, persistence, or error handling.
4. Use a graph path only to confirm a single source-target relation.
5. Open source files only for the symbols returned by the previous steps.

## Query construction

Extract the available values from the HU and substitute them into the playbook templates in
`docs/.agent-context/graphify-query-playbook.md`: `capability`, `endpoint`, `event`, `error`,
`service-a`, `service-b`. Do not include the HU identifier unless it is part of a contract or code.

## Anti-patterns

- Scanning every service when one owner candidate is strongly supported.
- Broad exploratory queries with no single technical question behind them.
- Re-querying in Phase B to "double check" something already recorded in the YAML.
- Treating a graph connection as proof of ownership; it is proof of a relation only.
