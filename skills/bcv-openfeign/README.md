# bcv-openfeign

Skill BCV/BACC para crear, revisar y diagnosticar clientes HTTP OpenFeign en servicios Java/Spring Boot.

## Cuándo usarlo

Usa este skill cuando el usuario pida:

- Crear un nuevo `@FeignClient` para integrarse a un servicio externo.
- Configurar `FeignConfig` (interceptor de headers o `ErrorDecoder`).
- Propagar `Authorization`, `X-Trace-Id`, `X-Span-Id` o `traceparent`.
- Manejar errores 4xx/5xx de servicios externos como excepciones de dominio.
- Mockear un cliente Feign en tests unitarios.
- Diagnosticar llamadas Feign que fallan o no propagan headers.

## Qué hace

- Genera la interfaz Feign con `base-url` y paths configurables (`interbank.ads.httpclient.feign.*`).
- Genera records de request/response.
- Configura el `RequestInterceptor` para propagar headers de trazabilidad y autenticación.
- Configura un `ErrorDecoder` tipado con saneamiento de PII.
- Cablea `@EnableFeignClients` en el módulo `-app`.
- Entrega el snippet de test con Mockito.

## Entradas esperadas

- Servicio BACC y API externa a integrar.
- Método(s): HTTP method, path, request/response y headers.
- Comportamiento de errores esperado (status codes).

## Salida esperada

- Interfaz Feign + records.
- `FeignConfig` (interceptor y/o decoder).
- Propiedades YAML.
- Test unitario con Mockito.

## Principios

- SDD: especificar el contrato externo antes de codificar.
- BMAD: Understand, Design, Build, Validate.
- URLs y paths configurables, nunca hardcodeados.
- Secretos desde Key Vault/env, nunca en el repo.
- Errores tipados (`ExternalServiceException`), nunca `FeignException` cruda.
- Saneamiento de PII en logs.

## Archivos clave

- `SKILL.md`
- `README.md`
- `evals/evals.json`
- `references/feign-client-pattern.md`
- `references/feign-error-decoder.md`
