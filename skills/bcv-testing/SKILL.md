---
name: bcv-testing
description: |
  Use this skill to generate, review or improve unit and integration tests for BCV/BACC Java Spring
  Boot services, following the ecosystem conventions: JUnit 5, Mockito (@ExtendWith(MockitoExtension.class)),
  AssertJ, @Mock/@InjectMocks, ArgumentCaptor for publishers/subscribers, @DataJpaTest for repositories,
  and JaCoCo coverage.
  Triggers: "generar tests", "pruebas unitarias", "unit test BCV", "cubrir con tests", "test adapter",
  "test publisher", "test subscriber", "test repository", "@DataJpaTest", "mockear puerto", "coverage JaCoCo",
  "aumentar cobertura", "test de caso de uso".
  Applies to all bcv-bacc-* services.
  Do NOT use for integration against real Azure Service Bus/Cosmos/SQL in CI (use mocks/fakes), or for
  contract testing of REST APIs (use bcv-openapi-design).
argument-hint: "component to test + type (use case / adapter / publisher / subscriber / repository) + scenarios"
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "testing"
  frameworks: ["Spec-Driven Development", "BMAD"]
---

# bcv-testing

## Language handling and output policy

See [references/language-policy.md](references/language-policy.md).

## Objective

Generate and review unit/integration tests for BACC services using the exact conventions already
present in the ecosystem: Mockito-first, no unnecessary `@SpringBootTest`, AAA structure, and
coverage aligned with JaCoCo.

## Scope

- Unit tests for use cases, adapters, mappers, publishers and subscribers.
- `@Mock` / `@InjectMocks` with `@ExtendWith(MockitoExtension.class)`.
- `ArgumentCaptor` for message/event assertions.
- `@DataJpaTest` for repository integration tests.
- FeignConfig / interceptor tests (header propagation).
- JaCoCo coverage and exclusion guidance.

## Conventions (detected in BACC)

| Aspect | Convention |
| --- | --- |
| Test framework | JUnit 5 (`org.junit.jupiter.api.*`) |
| Mocking | Mockito (`@ExtendWith(MockitoExtension.class)`, `@Mock`, `@InjectMocks`) |
| Assertions | AssertJ (`assertThat`) and/or JUnit `assert*` |
| Test class naming | `<ClassUnderTest>Test.java`, same package as production class |
| Method naming | `should...` or `test...` (both present in the codebase) |
| Structure | Given / When / Then (AAA) with short comments |
| Repositories | `@DataJpaTest` (rare; prefer mocking the port in unit tests) |
| Avoid | `@SpringBootTest` for unit tests, `@WebMvcTest` (not currently used) |

## Expected input

- Component (class/package) to test and its type (use case / adapter / publisher / subscriber / repository).
- Dependencies to mock and the scenarios (happy path, errors, edge cases).

## Expected output

1. Test class(es) following the package and naming conventions.
2. Mock setup (`@Mock` / `@InjectMocks`) and `when(...)` stubs.
3. `verify(...)` and assertion blocks.
4. For publishers/subscribers: `ArgumentCaptor` over `Message<T>`.
5. JaCoCo/coverage notes when relevant.

## Workflow

### SDD — Spec Driven Development

| Phase | Action |
| --- | --- |
| Especificar | Identify the component, its dependencies, and the scenarios to cover. |
| Validar | Confirm the component type and existing test style. Ask up to 3 questions. |
| Diseñar | Choose mock strategy (ports vs concrete dependencies) and assertion style. |
| Generar | Emit the test class(es) in the user's language. |
| Verificar | No real I/O in unit tests; assertions verify behavior, not implementation trivia. |

### BMAD — Build phase detail

1. Understand: component under test + dependencies + scenarios.
2. Design: mocks, stubs, captors, assertions.
3. Build: write the test following AAA.
4. Validate: run `mvn -q -Dtest=<TestClass> test` and check coverage.

## Mandatory patterns

Load `references/unit-test-patterns.md` before emitting code.

1. Use `@ExtendWith(MockitoExtension.class)`; avoid `@SpringBootTest` for unit tests.
2. Mock ports/interfaces with `@Mock`; inject into the class under test with `@InjectMocks`.
3. Use `when(...).thenReturn(...)` / `doThrow(...).when(...)` for stubs.
4. Use `verify(...)` and `ArgumentCaptor` to assert side effects (publishers, subscribers).
5. For `@Value` fields, inject via reflection in a small helper (the codebase already does this).
6. Prefer AssertJ `assertThat` for readability; JUnit `assertEquals` is also acceptable.
7. Cover happy path + at least one error/edge case per behavior.
8. No real network/DB/broker calls in unit tests; use fakes/mocks.

## Clarification questions (ask at most 3)

1. Which BACC service and component are you testing?
2. Is it a use case, adapter, publisher, subscriber or repository?
3. Which dependencies must be mocked, and what scenarios must be covered?

## Examples

**Example 1** — adapter test

Prompt: _Genera tests para CceServiceAdapter que llama a un puerto externo._

Response:

- `@ExtendWith(MockitoExtension.class)`, `@Mock` the port, `@InjectMocks` the adapter.
- Stub the port with `when(port.getCustomerSpecialStatus(any(), isNull())).thenReturn(response)`.
- Assert the returned domain and `verify(port).getCustomerSpecialStatus(...)`.

**Example 2** — publisher test

Prompt: _Test de PowersValidationPublisher._

Response:

- `@Mock` the `MessagePublisherRegistry` + `MessagePublisher`.
- Stub `registry.getPublisher("...")` to return the mock publisher.
- Use `ArgumentCaptor<Message>` and `verify(messagePublisher).publish(captor.capture())`.
- Assert the captured `subject`/`data`/`scheduledTime`.
- Cover the case where `publish` throws (assert it does not propagate).

**Example 3** — repository test

Prompt: _Test del repository de expedientes._

Response:

- Use `@DataJpaTest` for the repository, or mock `JpaRepository` in a unit test.
- Prefer mocking the port when testing the adapter; reserve `@DataJpaTest` for query verification.

## Evaluation

The skill output is valid when:

- [ ] Uses JUnit 5 + Mockito conventions (`@ExtendWith(MockitoExtension.class)`).
- [ ] Tests the component in the same package with `<Class>Test` naming.
- [ ] Mocks ports/interfaces, not real infrastructure.
- [ ] Asserts behavior (results + side effects) via `verify`/captors.
- [ ] Covers happy path and error/edge cases.
- [ ] No real network/DB/broker I/O in unit tests.
- [ ] Follows AAA (Given/When/Then) structure.

## When not to use this skill

1. The task is to test real messaging/DB/broker in CI (prefer fakes/mocks; this skill covers those).
2. The task is OpenAPI contract design (use `bcv-openapi-design`).
3. The task is writing production code, not tests.

## Reference loading policy

Load only what is needed:

1. `references/unit-test-patterns.md` — Mockito setup, adapter/publisher/subscriber examples, @Value injection.
2. `references/integration-test-patterns.md` — `@DataJpaTest` and JaCoCo coverage guidance.
