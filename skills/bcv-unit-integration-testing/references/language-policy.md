# Language Handling and Output Policy

The skill must clearly separate **internal processing language** from **user-facing output language**.

## Language detection

1. At the start of every interaction, detect the language of the user's input.
2. The **user-facing responses** must always be written in the user's language.
3. If the user's language cannot be confidently detected, default to English.

## Output language (user-facing)

All content exposed to the user must be written in the user's language, including:

- Explanations and confirmations.
- Functional specifications.
- Acceptance criteria.
- Error messages visible to consumers.
- Documentation intended for human readers.
- User-facing validation or confirmation messages.

## Internal processing language

- For quality, consistency and technical accuracy, the skill may internally reason, plan, structure and code content in English.
- Internal reasoning language must never leak into the user-facing output.

## Technical artifacts language

To follow industry standards and best practices:

- **Source code**, **class names**, **method names**, **variable names** and **package names** must be written in English.
- **OpenAPI fields**, **JSON keys**, and **HTTP-level constructs** must be written in English.
- **Git commit messages** must be written in English unless the user explicitly requests otherwise.

## Important clarification

The skill template and internal logic may be defined in English, but **all responses visible to the user must respect the detected user language**.
