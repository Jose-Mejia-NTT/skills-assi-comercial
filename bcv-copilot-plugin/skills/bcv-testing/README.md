# bcv-testing

Skill BCV/BACC para generar, revisar y mejorar tests unitarios e de integración en servicios Java/Spring Boot.

## Cuándo usarlo

Usa este skill cuando el usuario pida:

- Generar tests unitarios para casos de uso, adapters, publishers o subscribers.
- Aumentar la cobertura de un servicio (ej. `account-opening-reporting-service`).
- Testear un adapter que llama a un puerto externo.
- Verificar side effects de un publisher/subscriber con `ArgumentCaptor`.
- Testear un repository con `@DataJpaTest`.
- Aplicar convenciones JUnit 5 + Mockito + AssertJ.

## Qué hace

- Genera clases de test con `@ExtendWith(MockitoExtension.class)`, `@Mock` e `@InjectMocks`.
- Stubs con `when(...)` / `doThrow(...)` y verificaciones con `verify(...)`.
- Captura mensajes con `ArgumentCaptor<Message>` para publishers/subscribers.
- Inyecta campos `@Value` vía reflection en tests (patrón existente del código).
- Guía el uso de `@DataJpaTest` y JaCoCo.

## Entradas esperadas

- Componente (clase/paquete) a testear y su tipo.
- Dependencias a mockear.
- Escenarios (happy path, errores, edge cases).

## Salida esperada

- Clase(s) de test con el estilo AAA.
- Stubs, verificaciones y aserciones.
- Notas de cobertura JaCoCo.

## Principios

- SDD: identificar componente + escenarios antes de escribir.
- BMAD: Understand, Design, Build, Validate.
- Mockito-first; evitar `@SpringBootTest` en tests unitarios.
- Sin I/O real (red/DB/broker) en tests unitarios.
- Cubrir happy path + al menos un caso de error/edge por comportamiento.

## Archivos clave

- `SKILL.md`
- `README.md`
- `evals/evals.json`
- `references/unit-test-patterns.md`
- `references/integration-test-patterns.md`
