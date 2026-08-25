# Guardrails

1. **Evidence isolation.** Phase B writes the story exclusively from the emitted YAML. Anything not
   present in the YAML does not exist for the story. If Phase A discovered something worth keeping,
   go back and add it to the YAML first.
2. Graph evidence accelerates analysis but does not replace architecture judgment.
3. Run graph queries only in Phase A. Phase B performs no discovery: no graph artifacts, no queries,
   no source scanning. Missing evidence becomes an open item, not an improvised answer.
4. Separate `confirmed` vs `candidate` in every impacted dimension, in both artifacts.
5. Never present absolute conclusions with incomplete evidence.
6. Mark assumptions with `ASSUMED` and risks with `RISK`.
7. Do not perform business resolution: never invent capabilities or re-map ownership from scratch.
8. Do not modify source code, and do not generate implementation code.
9. Do not scan unrelated services when one owner candidate is strongly supported.
10. If output depends mostly on draft or stale artifacts, downgrade the gate to `REVIEW_REQUIRED`.
11. Carry Step 1 ambiguities and Phase A conflicts forward; never resolve them silently.
