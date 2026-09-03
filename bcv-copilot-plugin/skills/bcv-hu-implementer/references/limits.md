# Limits

These limits keep token consumption and risk under control.

## Read limits

| Source | Max |
| --- | --- |
| DHU file | Full file (expected < 500 lines) |
| Context file | Full file (expected < 300 lines) |
| Existing source file reads | 5 per repository |
| Lines per read | 40 max per read |

## Modification limits

| Action | Max per repository |
| --- | --- |
| Files modified | 10 |
| Files created | 5 |
| Tests created | 3 |
| Migrations created | 1 |

## Query limits

- Use `graphify` only to confirm existing structures, not to discover new ones.
- Max 2 graphify queries per repository.

## Safety limits

- Never modify more than one repository at a time in `apply` mode.
- If linter/tests fail, stop immediately.
- Never commit or push automatically.
