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

## Frontend / UI / Screen ownership gaps

This skill analyzes **backend microservices only**. It does not investigate frontend frameworks, screens, UI components, or presentation channels.

If the HU mentions a screen, channel, or UI flow (e.g., "canal BCW", "pantalla", "bandeja"), map it to the backend service that exposes the API, event, or data required by that channel.

### Do NOT generate gaps like

- "Who owns the BCW screen?"
- "Which frontend team implements the dropdown?"
- "Where is the UI component?"

### DO generate gaps like

- "Which backend API provides the registry office list to BCW?"
- "Which event/message carries the selected registry office to the channel?"
- "Which DTO/record must include the new field for the channel contract?"

If a screen mention cannot be mapped to any backend service, record the gap as:

```markdown
- **ID:** GAP-{NN}
- **Type:** technical
- **Blocking:** no
- **Description:** The HU references a channel/screen (BCW) but no backend API or event in the workspace serves it. Backend contract cannot be defined.
- **Suggested answer:** Confirm which backend service exposes data to BCW for this flow.
```

## Gap quality rules

- Be specific: state what is missing and why it matters.
- Do not invent information to fill a gap.
- If a gap is blocking, do not proceed to context generation.
- If a gap is non-blocking, mark it clearly so the next skill knows it is pending.
- Do not generate frontend/UI ownership gaps; this skill is backend-only.
