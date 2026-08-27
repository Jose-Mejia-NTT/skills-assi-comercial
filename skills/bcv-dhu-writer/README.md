# bcv-dhu-writer

Writes the final technical HU (DHU) Markdown document from a technical context file.

## When to use

- After `bcv-hu-context-analyzer` has generated `.context/hu-<code>.md`.
- When you need a specification-first technical story ready for implementation and QA.

## When NOT to use

- For repository investigation (use `bcv-hu-context-analyzer`).
- For business/functional resolution (use `bcv-business-resolution`).
- For implementation or test generation.

## Companion skill

- `bcv-hu-context-analyzer` — investigates the HU and produces the context file.

## Output format

The DHU follows the exact template defined by `ibk-hu-technical-refinement`.
See `references/output-template.md`.

## Main files

- `SKILL.md` — entry point and overview.
- `references/workflow.md` — step-by-step writing process.
- `references/output-template.md` — exact DHU format.
- `references/language-policy.md` — user-facing vs internal language rules.
