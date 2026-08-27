# Gap Handling

## Non-critical gap

Include it in `## 10. Gaps` with `Blocking: No`.

The companion skill `bcv-dhu-writer` will move it to "Next Steps / Pending Decisions" in the final DHU.

## Critical / blocking gap

Stop the skill and ask the user for clarification if any of the following occurs:

- No injection point can be identified.
- The HU has no acceptance criteria.
- No repository in the workspace seems related to the HU.
- A required external system (catalog, master service) is completely missing and the HU cannot be analyzed without it.

Response format:

```text
Cannot generate technical context because:
- <gap 1>
- <gap 2>

Please clarify these points before continuing.
```

Do not write an empty context file or an incomplete DHU.

## Gap quality rules

- Be specific: state what is missing and why it matters.
- Do not invent information to fill a gap.
- If a gap is blocking, do not proceed to context generation.
- If a gap is non-blocking, mark it clearly so the next skill knows it is pending.
