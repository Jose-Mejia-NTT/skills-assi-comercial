# Layer dependency rules

Use this reference to enforce clean/hexagonal architecture boundaries in BCV services.

## Module layout

```
bcv-bacc-<service>-service/
├── -core/    # domain, use cases, port.in, port.out (no infrastructure deps)
├── -input/   # REST controllers, subscribers, DTOs, mappers (entry points)
├── -output/  # persistence adapters, publishers, Feign clients (driven adapters)
└── -app/     # launcher, config, bean wiring (UseCaseConfig)
```

## Allowed vs forbidden dependencies

| From | To | Allowed? |
| --- | --- | --- |
| `input` | `core` | ✅ |
| `output` | `core` | ✅ |
| `app` | all | ✅ (wiring only) |
| `core` | `input` | ❌ |
| `core` | `output` | ❌ |
| `input` | `output` | ❌ |
| `output` | `input` | ❌ |

## Violation examples and fixes

### `core` importing a Web annotation

```java
// ❌ core/domain/Expedient.java
import org.springframework.web.bind.annotation.RestController;
```

Fix: move any web/transport concern to `input`; core stays framework-agnostic.

### `core` importing JPA

```java
// ❌ core/domain/Expedient.java
import jakarta.persistence.Entity;
```

Fix: keep a pure domain model in core; map to a JPA entity in `output/database/entity`.

### `input` importing an `output` adapter directly

```java
// ❌ input/controller/ExpedientController.java
import pe.interbank.bcv.<service>.out.database.repository.ExpedientRepository;
```

Fix: controller depends on a `port.in` command/query interface; wiring is done in `app`.

### Use case with Spring stereotype

```java
// ❌ core/usecase/CreateExpedientService.java
@Service
public class CreateExpedientService { ... }
```

Fix: plain Java class; register the bean in `app/config/UseCaseConfig`.

## Ports and adapters conventions

```java
// core/port/in/command/CreateExpedientInCommandPort.java
public interface CreateExpedientInCommandPort {
    void create(ExpedientCommand command);
}

// core/port/out/ExpedientPersistenceOutPort.java
public interface ExpedientPersistenceOutPort {
    void save(Expedient domain);
}

// core/usecase/CreateExpedientService.java  (plain Java, depends on out port)
public class CreateExpedientService implements CreateExpedientInCommandPort {
    private final ExpedientPersistenceOutPort outPort;
    public CreateExpedientService(ExpedientPersistenceOutPort outPort) {
        this.outPort = outPort;
    }
    // ...
}

// output/persistence/ExpedientPersistenceAdapter.java  (implements out port)
@Component
public class ExpedientPersistenceAdapter implements ExpedientPersistenceOutPort {
    // JPA repository here
}
```

## Detection heuristics (audit)

- `grep -rn "jakarta.persistence" <core>` → infrastructure leak into core.
- `grep -rn "org.springframework.web" <core>` → transport leak into core.
- `grep -rn "out\." <input>` → input depending on output.
- `grep -rn "@Service\|@Component" <core/usecase>` → stereotypes in use cases.
- `grep -rn "import.*repository" <input>` → input bypassing ports.

## Guardrails

1. Refactors must not change behavior; only structure and imports.
2. Fix dependency direction before extracting new modules.
3. Keep public API and message contracts stable unless explicitly requested.
4. Verify `mvn -q -DskipTests package` after each move.
