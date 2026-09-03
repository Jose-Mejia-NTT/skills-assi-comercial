# Profiles and Configuration Boundaries

Use this reference to separate runtime configuration responsibilities.

## Profile strategy

Common BCV profiles:

- `local`
- `dev`
- `uat`
- `stg`
- `prd`

Keep behavior predictable across profiles:

- `local`: developer defaults, local stubs/mocks when needed
- `dev/uat/stg/prd`: environment-managed secrets and real integrations

## `bootstrap.yml` vs `application.yml`

Use `bootstrap.yml` for:

- Config Server bootstrap properties
- Azure Key Vault property source setup
- secret key declarations used by startup

Use `application.yml` for:

- Application runtime settings (server port, feature flags, logging)
- non-secret business properties
- profile-specific toggles

## Anti-patterns

- Storing secret values directly in `application.yml`.
- Mixing bootstrap-only concerns into runtime application sections.
- Different property names for the same integration across profiles without migration plan.
