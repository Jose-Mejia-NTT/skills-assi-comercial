# error-handling-patterns

Use project-local conventions; do not force a single global pattern.

Core service layer:
1. Preserve existing DomainException handling style.
2. Preserve existing UnexpectedException handling style.
3. Keep logging level and message style consistent with existing use cases.

Output adapter layer:
1. If project uses DatabaseIOException (or equivalent), keep using it.
2. If project uses another strategy, mirror that strategy.
3. Avoid introducing a new exception abstraction unless required by project conventions.
