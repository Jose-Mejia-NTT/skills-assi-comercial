# Extraction rules

Extract only concepts supported by the input or catalog:

- `systems`: named business or external systems.
- `entities`: business objects, roles and important data elements.
- `actions`: business verbs and state transitions.
- `events`: business events, triggers and outcomes.
- `rules`: thresholds, eligibility, exclusions, defaults and invariants.
- `scope`: explicit inclusions and exclusions.

Normalize spelling and aliases without changing business meaning. Preserve original terms when
normalization could lose meaning.

## Extraction patterns

Use these patterns to recognize concepts in the HU/HAB:

| Pattern                          | Example                                                               | Concept                   |
| -------------------------------- | --------------------------------------------------------------------- | ------------------------- |
| "cuando X ocurre, entonces Y"    | "cuando el pago es rechazado, se notifica al cliente"                 | evento + acción           |
| "el sistema debe validar que..." | "el sistema debe validar que el cliente sea mayor de edad"            | regla                     |
| "solo aplica para..."            | "solo aplica para cuentas activas"                                    | scope                     |
| "quiero... para que..."          | "quiero consultar el saldo para que el cliente conozca su disponible" | actor + acción + objetivo |
| "dado que..." / "dada una..."    | "dada una cuenta en estado bloqueado"                                 | precondición / regla      |
