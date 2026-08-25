# Guardrails

1. Do not inspect or modify source files as part of this skill.
2. Do not read anything under `graphify-out/` (`GRAPH_REPORT.md`, `graph.json`, `graph.html`),
   do not run graph queries or graph path commands, and do not load
   `docs/.agent-context/graphify-query-playbook.md`. Graph evidence belongs to Step 2 only.
3. Do not generate a technical story, implementation plan or production code.
4. Do not hide missing catalog entries or unsupported terms.
5. Mark assumptions as `ASSUMED` and risks as `RISK`.
6. Ask up to 3 focused clarification questions only when they can change the candidate mapping.
7. Keep the output concise and suitable for handoff to another skill.
8. Never output Step 2 conclusions (`impacted_apis`, `impacted_persistence`, `impacted_events`) as facts.
9. Do not assert synchronous/asynchronous integration paths unless they are explicitly in provided
   business input or reference catalog.
