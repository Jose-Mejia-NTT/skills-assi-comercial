# Flat → hexagonal migration checklist

Use this reference to migrate a legacy flat service (e.g. `bcv-bacc-service-point-service`) to a
clean/hexagonal module split. Execute incrementally; build and test after every step.

## Preconditions

- [ ] Full `git` baseline (branch) created.
- [ ] Current architecture classified (flat vs layered).
- [ ] Public REST API and message contracts inventoried (they must not change).
- [ ] Existing tests captured so behavior can be re-verified.

## Step 1 — Inventory the flat service

Flat packages typically found in `service-point-service`:

```
pe.interbank.bcv.baccservicepoint/
├── client/       # Feign clients
├── config/       # @Configuration, @ConfigurationProperties
├── controller/   # REST controllers
├── domain/       # domain/entities (mixed with JPA)
├── dto/          # request/response DTOs
├── error/        # exception handlers
├── facade/       # orchestration facades
├── mapper/       # MapStruct mappers
├── publisher/    # Service Bus publishers
├── repository/   # JPA repositories
├── service/      # @Service business logic
├── subscriber/   # Service Bus subscribers
├── util/         # helpers
└── validator/    # validators
```

Map each package to its hexagonal home:

| Flat package               | Hexagonal home                                         |
| -------------------------- | ------------------------------------------------------ |
| `domain`                   | `-core/domain`                                         |
| `service` (business rules) | `-core/usecase` (+ `-core/port/in` + `-core/port/out`) |
| `facade` (orchestration)   | `-core/usecase` or `-input` command/query services     |
| `controller`               | `-input/controller`                                    |
| `dto` + `mapper`           | `-input/dto` + `-input/mapper`                         |
| `repository`               | `-output/database/repository`                          |
| `client`                   | `-output/client`                                       |
| `publisher`                | `-output/broker/publisher`                             |
| `subscriber`               | `-input/broker/subscriber`                             |
| `config` (app wiring)      | `-app/config`                                          |
| `error`                    | `-input/error` (advice)                                |
| `util`, `validator`        | keep with the layer that uses them                     |

## Step 2 — Create `-core`

1. Create the `-core` module and parent POM entry.
2. Move pure domain classes (no Spring/JPA annotations) into `-core/domain`.
3. Extract use cases from `service`/`facade` into `-core/usecase` as plain Java classes.
4. Define `-core/port/in` and `-core/port/out` interfaces.

## Step 3 — Create `-output`

1. Create the `-output` module.
2. Move `repository`, `client`, `publisher` into `-output`.
3. Wrap JPA entities in `-output/database/entity`; keep domain pure in core.
4. Implement `port.out` interfaces in adapters.

## Step 4 — Create `-input`

1. Create the `-input` module.
2. Move `controller`, `subscriber`, `dto`, `mapper` into `-input`.
3. Controllers depend on `port.in`, not on concrete services.

## Step 5 — Create `-app` and wire

1. Create the `-app` module; move the launcher and `config` here.
2. Register use case beans in `UseCaseConfig`.
3. Keep `spring-boot-maven-plugin` executable only in `-app`.

## Step 6 — Verify

- [ ] `mvn -q -DskipTests package` succeeds.
- [ ] Unit/integration tests pass.
- [ ] Layer dependency rules hold (`core` imports nothing from input/output).
- [ ] REST paths and message queue/topic names unchanged.
- [ ] No secrets hardcoded during the move.

## service-point-service specific notes

- Parent is `bcv-commons-pomparent` 2.0.0 and Java 17 — do NOT bump to ADS parent/Java 21 during a
  pure architecture migration unless explicitly requested.
- 255+ Java files: migrate one bounded context at a time (e.g. `service-point`, then `service-tray`).
- `@ConfigurationProperties` (`FlagProperties`, `TrayProperties`, `StorageProperties`) move to
  `-app/config` or `-output` depending on what consumes them.
