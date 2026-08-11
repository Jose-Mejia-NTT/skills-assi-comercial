---
name: bcv-java-spring-boot
description: |
  Use this skill for BCV Java/Spring Boot engineering tasks across all BCV projects: ADS BOM and parent setup,
  multi-module Maven structure, executable app-module conventions, Spring profiles, bootstrap/application
  configuration, Spring Cloud Config + Azure Key Vault secret handling, startup troubleshooting, and safe
  integration patterns for external dependencies.
  Triggers: "Spring Boot BCV", "ADS BOM", "bcv parent", "multi-module", "bootstrap.yml",
  "application.yml", "Config Server", "Key Vault", "BcvDisbBusinessService", "DocumentManagementAppLauncher",
  "ExpedientManagementAppLauncher", "MainApplication", "no arranca en local", "startup error".
  Applies to: bcv-disb-business-service, bcv-h2h-document-management-service,
  bcv-h2h-expedient-management-service, bcv-h2h-integration-service.
  Do NOT use for: detailed JPA modeling (use bcv-spring-data-jpa-sql-server), Service Bus implementation
  (use bcv-azure-service-bus), or domain-specific rules (use domain skills).
argument-hint: "project + goal (e.g., add module, fix startup, configure profiles, secure secrets)"
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "backend"
  frameworks: ["Spec-Driven Development", "BMAD"]
---

# bcv-java-spring-boot

## Language rules

- Internal processing, generated code and structural reasoning: English.
- Response to the user: the language of the user's initial message (default: Spanish).
- Preserve class names, artifactIds, property keys, and package names in original form.

## Objective

Provide BCV-specific Spring Boot guidance and production-safe snippets that match existing project conventions:
BOM/parent alignment, module boundaries, configuration externalization, secure secrets handling, and startup diagnostics.

## Scope

- Parent/BOM setup (`ads-spring-boot-dependencies`, `bcv-commons-starter-parent`)
- Multi-module Maven conventions (`-core`, `-input`, `-output`, `-app`)
- Executable module rules (`spring-boot-maven-plugin` placement)
- `bootstrap.yml` vs `application.yml` usage
- Spring profiles (`local`, `dev`, `uat`, `stg`, `prd`)
- Config Server + Azure Key Vault secret resolution
- Local startup troubleshooting and safe defaults
- Mock-first strategy when external dependencies are unavailable

## Inputs

Natural language requests such as:

- "Como agrego un nuevo modulo Maven en un proyecto BCV con ADS BOM?"
- "My service fails at startup when Config Server is down."
- "How should I split app/core/out modules for a new BCV service?"
- "Necesito configurar perfiles y secretos sin hardcodear credenciales."

## Expected output

1. Project variant identified (which BCV project pattern applies).
2. Minimal dependency/module changes (POM snippets only if needed).
3. Configuration snippets (`bootstrap.yml`, `application.yml`, env vars).
4. Secure secret strategy (Key Vault, no hardcoded secrets).
5. Mock-first recommendation if external systems are unavailable.
6. Validation checklist and startup troubleshooting steps.

## Workflow

### SDD - Spec Driven Development

| Phase | Action |
| --- | --- |
| Especificar | Parse intent: setup, refactor, startup fix, or configuration hardening. |
| Validar | Confirm project type and missing constraints. Ask up to 3 clarifying questions. |
| Disenar | Select BCV-compatible structure, BOM, profile strategy, and secret flow. |
| Generar | Return smallest complete change set (POM/config/code) in user language. |
| Verificar | Validate security, module boundaries, and run-readiness checks. |

### BMAD - Build phase detail

1. Understand: detect project flavor and operational constraint.
2. Design: map to BCV conventions (modules, parent, profiles, secrets).
3. Build: produce targeted snippets and exact placement.
4. Validate: confirm with checklist before final response.

## Mandatory patterns

1. Keep one executable module (`-app`) and avoid accidental boot plugin execution in library modules.
2. Never hardcode credentials, tokens, webhook URLs, or Service Bus connection strings.
3. Put environment and secret wiring in `bootstrap.yml` when using Config Server/Key Vault.
4. Keep `application.yml` focused on runtime behavior (ports, feature flags, logging, profiles).
5. If external dependency is unavailable (Config Server, DB, broker, third-party API), propose mock/stub strategy first.
6. Match package conventions under `pe.interbank.bcv.*` when applicable.
7. Favor minimal diffs: adjust only required modules/properties.

## Clarification questions (ask at most 3)

1. Which BCV project are you working on?
2. Is your goal setup, migration, or startup troubleshooting?
3. Which external dependency is currently unavailable (if any)?

## Reference loading map

Load only the reference needed for the user request:

| Topic | Reference |
| --- | --- |
| Parent/BOM and module conventions | `./references/bom-and-modules.md` |
| Profiles and config boundaries | `./references/profiles-and-config.md` |
| Config Server + Key Vault secret flow | `./references/config-server-keyvault.md` |
| Startup failures and recovery checklist | `./references/startup-troubleshooting.md` |

## Token-efficiency rules

- Keep `SKILL.md` under 500 lines.
- Prefer checklists/tables over long prose.
- Return only the relevant snippet for the identified variant.
- Avoid duplicating examples already covered by references.

## Examples

### Example 1

Prompt: "Como agrego un nuevo modulo Maven en un proyecto BCV con ADS BOM?"

Expected behavior:

- Detect parent/BOM pattern.
- Add `<module>` in parent POM.
- Create module POM inheriting parent.
- Keep boot plugin runnable only in app module.
- Add dependency wiring only where required.

### Example 2

Prompt: "El servicio no levanta en local si falla Config Server. Que reviso?"

Expected behavior:

- Verify profile (`local`) and fail-fast settings.
- Confirm Key Vault environment variables and secret names.
- Suggest local fallback strategy and mock endpoints.
- Provide startup diagnosis checklist.

### Example 3

Prompt: "How do I configure secrets securely for BCV Spring Boot apps?"

Expected behavior:

- Show `bootstrap.yml` Key Vault/Config Server wiring.
- Keep secret values externalized through env vars.
- Warn against hardcoded values in git.
- Provide minimum verification steps.

## Evaluation

Output is valid when:

- [ ] Identifies the correct BCV variant and module pattern.
- [ ] Uses proper parent/BOM guidance for the project context.
- [ ] Enforces secure secret handling (Key Vault/env, no hardcoding).
- [ ] Distinguishes `bootstrap.yml` and `application.yml` responsibilities.
- [ ] Applies mock-first when external dependencies are unavailable.
- [ ] Includes a practical troubleshooting or verification checklist.

## Delivery contract

When the user asks for implementation help, return:

1. Files to touch (path + intent).
2. Minimal snippets to add/change.
3. Risk notes (security/startup/compatibility).
4. Verification commands or checks.
