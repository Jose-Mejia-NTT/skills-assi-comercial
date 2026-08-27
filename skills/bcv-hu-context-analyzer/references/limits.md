# Token Budget and Limits

These limits are mandatory and must be stated explicitly in the response.

## graphify queries

| Service type | Max queries per service | Default `--max` | Max `--max` |
| --- | --- | --- | --- |
| Primary | 2 | 5 | 10 |
| Secondary | 1 | 5 | 10 |
| Omitted | 0 | — | — |

- Always start with `--max 5`.
- Increase to `--max 10` only if 5 results do not reveal the injection point.
- Never use `graphify query "*"` or `--max` greater than 10.

## Code reads

| Metric | Limit |
| --- | --- |
| Total reads per session | 3 |
| Lines per read (normal) | 40 |
| Lines per read (exceptional) | 80 |

Report in every response: **Reads used: N/3**.

## README reads

- **Prefer** `docs/.agent-context/service-map.md` (max 30 lines).
- Only fall back to README summary/scope if `.agent-context/` is missing.
- Maximum 30 lines per README.
- Do not read full READMEs.

## graphify build

- Use `graphify <repo> --code-only` to avoid LLM extraction of docs.
- Build graphs only for primary and secondary services.
- Reuse existing `graphify-out/` when available.

## What NOT to do

- Do not load the full `graphify` skill.
- Do not search synonyms if the exact concept returns no results.
- Do not read libraries or external dependencies.
- Do not generate code, tests, or migrations.
