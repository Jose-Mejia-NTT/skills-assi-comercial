# Integration test patterns (BACC)

Use this reference for repository tests and coverage guidance.

## Repository tests (@DataJpaTest)

`@DataJpaTest` is used sparingly in BACC. Reserve it for verifying actual query semantics;
prefer mocking the port when testing an adapter.

```java
package pe.interbank.bcv.baccaccountopeningreporting.out.database.repository;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
class ExpedientRepositoryTest {

    @Autowired
    private ExpedientRepository repository;

    @Test
    void findByExpedientNumber_shouldReturnExpedient() {
        // given
        var entity = ExpedientEntity.builder()
                .expedientNumber("EXP-001")
                .status("PENDING")
                .build();
        repository.save(entity);

        // when
        Optional<ExpedientEntity> found = repository.findByExpedientNumber("EXP-001");

        // then
        assertThat(found).isPresent();
        assertThat(found.get().getStatus()).isEqualTo("PENDING");
    }
}
```

Notes:

- BACC tables use `ddl-auto=none`; `@DataJpaTest` may require an embedded DB or a test
  `application.yml` datasource. Confirm the service's existing test config before using it.
- Prefer `@Mock` + `@InjectMocks` for adapter tests (the dominant BACC style).

## JaCoCo coverage

JaCoCo is configured in the parent `pom.xml` (`jacoco-maven-plugin`, `0.8.12`):

```xml
<property>
  <name>jacoco.plugin.version</name>
  <value>0.8.12</value>
</property>
```

- Run: `mvn -q clean test jacoco:report`.
- Aim for the service policy (typically 70-80% line coverage).
- Exclude generated mappers (MapStruct `*MapperImpl`) and DTO/record getters from coverage when the
  policy allows.
- `account-opening-reporting-service` currently has no visible tests: prioritize use cases, adapters
  and publishers/subscribers there.

## What NOT to test with real infrastructure

- Do not spin up real Azure Service Bus, Cosmos DB or SQL Server in unit/integration tests.
- Use mocks for ports and fakes for persistence when possible.
- Contract/API behavior belongs to `bcv-openapi-design`; wire-level integration belongs to CI/e2e.

## Checklist

- [ ] Repository tests use `@DataJpaTest` only when query semantics must be verified.
- [ ] Adapter tests mock the port with Mockito.
- [ ] JaCoCo report is generated and coverage meets the service policy.
- [ ] Generated mappers are excluded from coverage where allowed.
- [ ] No real external I/O in tests.
