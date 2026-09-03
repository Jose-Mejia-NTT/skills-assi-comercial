# Data masking

When `bcv.observability.data-masking.enabled: true`, the `bcv-commons-observability` library masks sensitive values in logs based on built-in rules.

## Enable masking

```yaml
bcv:
  observability:
    data-masking:
      enabled: true
```

## Default masked fields

The library typically masks fields such as:

- `documentNumber` / `documento` / `dni` / `ruc`
- `accountNumber` / `numeroCuenta`
- `phoneNumber` / `telefono`
- `email` (partial mask)
- `password`, `token`, `secret`

Always check the current version of `bcv-commons-observability` for the exact rule list.

## Rules of thumb

- Keep PII out of `log.info`/`log.debug` raw messages; rely on the masking rules for unavoidable fields.
- Do not log full request/response bodies that may contain passwords, tokens or account numbers unless they are explicitly allowed.
- In `bcv-disb-business-service` the masking layer is provided by `bcv-commons-observability`.
- In `bcv-h2h-integration-service` it works alongside the WebFlux context.

## Example

```text
Before: customerDocument=12345678
After:  customerDocument=********
```

```text
Before: accountNumber=0011001234567890123
After:  accountNumber=**************0123
```

## Custom masking rules

If you need to mask a new field, request an update of the masking rules in the `bcv-commons-observability` library. Do not implement a parallel masking mechanism in project code.

## Anti-patterns

```java
// BAD: raw PII in log message
log.info("Customer DNI: {}", request.getDni());

// GOOD: rely on masking or avoid PII in logs
log.info("Processing request for customer");
```

## Verification

Run a local test that logs a request object and confirm the output:

```java
@Slf4j
@SpringBootTest
class DataMaskingTest {

    @Test
    void shouldMaskDniInLogs() {
        var request = PaymentPromiseRequestDto.builder()
            .dni("12345678")
            .build();
        log.info("Request: {}", request);
        // Check console/log file: dni value should be masked
    }
}
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| PII still visible | `data-masking.enabled: false` | Set to `true` |
| Field not masked | Rule not in library | Request rule update |
| Masking in test only | Test uses different `application.yml` | Add property to `application-test.yml` |
