---
name: bcv-hexagonal-architecture
description: |
    Use this skill when the user wants to add a new feature, use case, or endpoint to a
    BCV H2H service that follows a hexagonal architecture pattern (core/input/output/app).
    Triggers: "add use case", "new endpoint", "new feature", "agregar caso de uso",
    "nuevo endpoint", "nueva funcionalidad", "necesito que el servicio pueda...",
    "quiero agregar...", or any natural language description of a new capability in
    a BCV H2H domain.
    Do NOT use for: modifying existing use cases, generating tests (use bcv-pitest-mutation),
    generating OpenAPI (use bcv-openapi-design), or Azure Service Bus events (use bcv-azure-service-bus).
applyTo: "**"
---

# bcv-hexagonal-architecture

# Objetivo

Generate a complete hexagonal vertical slice for a new use case in
the current BCV H2H project, following the existing project patterns exactly.
One skill invocation = one use case = one complete slice.

## Project module structure

```
*-core/    ← Domain: entities, value objects, use case interfaces, service implementations
*-input/   ← Driving adapter: REST controllers, DTOs, mappers, application services
*-output/  ← Driven adapter: persistence adapters, repositories, entities, mappers
*-app/     ← Wiring: @Bean configuration in UseCaseConfig
```

Base package: discover from project source tree (do not hardcode).

## Project alignment (mandatory before generation)

Before generating code, inspect the current repository and infer:

1. Base package root used by Java sources.
2. Exact module names for `*-core`, `*-input`, `*-output`, `*-app`.
3. Existing package conventions for:
    - `core.port.in` (with or without domain subfolder)
    - `output` adapter path (for example `out/database/adapter` or `out/adapter`)
    - controller path style and API naming
4. Existing exception strategy in persistence adapters:
    - if the project uses `DatabaseIOException` (or equivalent), follow it.
    - if not, preserve current project style.

If any of the above cannot be inferred confidently, ask up to 3 clarifying questions.

# Input esperado

Natural language description of the new feature. Examples:
- "Necesito que el servicio pueda reabrir un expediente anulado"
- "Add an endpoint to reassign an expedient to another user with justification"
- A DHU section with acceptance criteria and business rules

# Output esperado

For a use case named `{Xxx}`:

| File | Action | Module |
|------|--------|--------|
| `core/port/in/{domain}/{Xxx}UseCase.java` | CREATE | core |
| `core/usecase/{Xxx}Service.java` | CREATE | core |
| `core/port/out/{Xxx}PersistencePort.java` | CREATE if new persistence needed | core |
| `input/app/in/dto/request/{Xxx}RequestDto.java` | CREATE | input |
| `input/app/in/dto/response/{Xxx}ResponseDto.java` | CREATE if response needed | input |
| `input/app/in/mapper/{Xxx}ControllerMapper.java` | CREATE or UPDATE | input |
| `input/app/application/{Xxx}CommandService.java` or `QueryService` | CREATE or UPDATE | input |
| `input/app/in/controller/{Existing}Controller.java` | ADD METHOD | input |
| `output/<project-output-adapter-path>/{Xxx}PersistenceAdapter.java` | CREATE or UPDATE | output |
| `app/config/UseCaseConfig.java` | ADD @Bean | app |

# Flujo de trabajo

## SDD — Spec Driven Development

| Phase | Action |
|-------|--------|
| **Especificar** | Parse the user input (natural language or DHU) and produce an internal structured spec: action verb, entity, inputs, outputs, business rules, Command vs. Query. |
| **Validar** | Verify the spec is complete. If missing critical information, ask at most 3 clarifying questions (see section below). Score must pass before generating. |
| **Generar** | Only when the spec is approved, generate all files of the hexagonal slice in order: core → input → output → app. |
| **Verificar** | Check layer dependency rules and `UseCaseConfig` registration. Report any violation before delivering the output. |

## BMAD — Build phase detail

### Understand
1. Parse the user request — identify: action verb, domain entity, required inputs, expected output, business rules.
2. Map to the existing domain entities and value objects already present in the current project.
3. Identify which existing ports/adapters can be reused vs. what needs to be created.
4. Determine if the use case is a Command (mutates state) or Query (reads state).

### Design
5. Name the use case following the convention: `{Action}{Entity}UseCase` → `{Action}{Entity}Service`.
6. Define the port/in interface method signature using existing value objects when possible.
7. Define required DTOs (record classes) and mapper methods.
8. Identify the HTTP method and path following the existing controller pattern used by the project.

### Build
9. Generate files in order: core → input → output → app (wiring).
10. Apply all mandatory patterns (see Rules section).

### Validate
11. Verify no layer imports from a layer it should not know about.
12. Verify `UseCaseConfig` has the new @Bean registered.
13. Verify controller uses `@ObservableOperation` on the new method.

## Mandatory patterns

### core — UseCase interface
```java
package <project-base-package>.core.port.in.<domain-or-root>;

public interface {Xxx}UseCase {
    {ReturnType} {methodName}({params});
}
```

### core — Service implementation
```java
@Slf4j
@RequiredArgsConstructor
public class {Xxx}Service implements {Xxx}UseCase {

    // inject only port/out interfaces, never adapters or Spring beans
    private final {Yyy}PersistencePort {yyy}Port;

    @Override
    public {ReturnType} {methodName}({params}) {
        try {
            // business logic
        } catch (DomainException de) {
            log.warn("...", de.getMessage());
            throw de;
        } catch (Exception e) {
            log.error("...", e.getMessage());
            throw new UnexpectedException(0, e);
        }
    }
}
```

> Use cases are plain Java — NO `@Service`, NO `@Component`. Wired exclusively via `UseCaseConfig`.

### input — Controller method
```java
@ObservableOperation
@PostMapping(path = "/{id}/action",
             consumes = MediaType.APPLICATION_JSON_VALUE,
             produces = MediaType.APPLICATION_JSON_VALUE)
public ResponseEntity<{Xxx}ResponseDto> {methodName}(
        @PathVariable Long id,
        @RequestBody {Xxx}RequestDto requestDto) {
    // delegate to CommandService or QueryService
}
```

> Always add `@Operation`, `@ApiResponses` (200, 400, 500) and `@ObservableOperation`.

### input — DTOs
```java
// Use Java records. No lombok on DTOs.
public record {Xxx}RequestDto(
    @JsonProperty("field") String field
) {}
```

### output — Persistence adapter
```java
@Component
@RequiredArgsConstructor
@Slf4j
public class {Xxx}PersistenceAdapter implements {Xxx}PersistencePort {
    private final {Xxx}Repository repository;

    @Override
    public void {methodName}({params}) {
        // follow current project persistence exception strategy
    }
}
```

### app — UseCaseConfig bean registration
```java
@Bean
public {Xxx}UseCase {xxxUseCase}({dependencies}) {
    return new {Xxx}Service({dependencies});
}
```

## Clarification questions (ask only if missing)

Ask at most 3 questions before generating:

1. What is the action and domain entity? (if not clear from the request)
2. Should this mutate state (Command) or only read (Query)?
3. Does it require a new persistence operation or can it reuse an existing port?

If the user provides a DHU, extract answers from criteria sections and rules before asking.

## Layer dependency rules (enforce strictly)

```
input  → core (allowed)
output → core (allowed)
core   → input (FORBIDDEN)
core   → output (FORBIDDEN)
input  → output (FORBIDDEN)
app    → all (allowed, wiring only)
```

# Reglas de lenguaje

- Internal processing and generated code: English.
- Response to user: language of the user's initial message (default: Spanish).

# Reglas de token-efficiency

- Do not repeat patterns already shown above.
- Generate all files in a single response, grouped by module.
- Use `// ... existing methods` to indicate unchanged code sections when modifying a file.

# Evaluación

The skill output is considered valid when:

- [ ] All files of the slice are generated (port/in, service, DTOs, mapper, controller method, adapter, @Bean).
- [ ] No layer imports a layer it is not allowed to depend on.
- [ ] `UseCaseConfig` registers the new use case as a `@Bean`.
- [ ] The controller method has `@ObservableOperation`, `@Operation` and `@ApiResponses` (200, 400, 500).
- [ ] The service class has NO `@Service` / `@Component` — it is plain Java.
- [ ] DTOs are Java `record` classes — no Lombok.
- [ ] The persistence adapter follows the existing project exception strategy (for example `DatabaseIOException` where applicable).
- [ ] The service wraps domain errors in `DomainException` and unexpected errors in `UnexpectedException`.
