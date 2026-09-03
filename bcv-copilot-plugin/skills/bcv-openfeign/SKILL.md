---
name: bcv-openfeign
description: |
  Use this skill for any BCV/BACC Java Spring Boot question about OpenFeign HTTP clients: creating or
  reviewing a Feign client interface, FeignConfig (RequestInterceptor + ErrorDecoder), header
  propagation (Authorization, X-Trace-Id, traceparent), configurable base-url/path properties,
  request/response records, @EnableFeignClients wiring, error handling for external services, or
  mocking Feign clients in tests.
  Triggers: "crear cliente Feign", "nuevo @FeignClient", "Feign error decoder", "propagar headers",
  "RequestInterceptor", "interbank.ads.httpclient.feign", "external service call", "Feign 404/500",
  "circuit breaker Feign", "mockear Feign client".
  Applies to all bcv-bacc-* services.
  Do NOT use for Azure Service Bus messaging (use bcv-azure-service-bus), REST API contract design
  (use bcv-openapi-design), or generic Spring Cloud OpenFeign tutorials.
argument-hint: "service + goal (create client / fix error handling / propagate headers) + external API"
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "backend"
  frameworks: ["Spec-Driven Development", "BMAD"]
---

# bcv-openfeign

## Language handling and output policy

See [references/language-policy.md](references/language-policy.md).

## Objective

Create, review and troubleshoot OpenFeign clients in BACC services, following the exact
conventions already present in the ecosystem: configurable base-url/path via
`interbank.ads.httpclient.feign.*`, header propagation through a `RequestInterceptor`, and
typed error handling through a custom `ErrorDecoder`.

## Scope

- Feign client interfaces (`@FeignClient`) in `out.client` / `output.client`.
- Request/response Java records in `out.client.record` / `output.client.record`.
- `FeignConfig` (`RequestInterceptor` + `ErrorDecoder`) in `out.client.config`.
- `@EnableFeignClients` wiring in the `-app` module.
- Header propagation (`Authorization`, `X-Trace-Id`, `X-Span-Id`, `traceparent`).
- Error mapping (4xx/5xx → domain exception) and PII sanitization in logs.
- Mocking Feign clients in unit tests.

## Project conventions (BACC)

| Concern | Location |
| --- | --- |
| Client interface | `out.client.*` (hexagonal) / `client.*` (legacy `service-point-service`) |
| Client DTOs (records) | `out.client.record.*` |
| FeignConfig | `out.client.config.FeignConfig` |
| Error decoder | `out.client.config.*ErrorDecoder` (or `app/config/error/external/feign`) |
| Enable wiring | `app/config/FeignConfiguration` with `@EnableFeignClients` |

## Expected input

- Target BACC service and the external API to integrate.
- Endpoint method(s): HTTP method, path, request/response shape, headers.
- Error behavior: expected status codes and how to surface them.

## Expected output

1. The Feign client interface with configurable path properties.
2. `FeignConfig` updates (interceptor and/or error decoder).
3. Request/response records.
4. `application.yml` / `bootstrap.yml` property entries.
5. A Mockito test snippet mocking the client.

## Workflow

### SDD — Spec Driven Development

| Phase | Action |
| --- | --- |
| Especificar | Parse the external API: method, path, query/body/headers, success and error responses. |
| Validar | Confirm the service and existing Feign conventions. Ask up to 3 questions if needed. |
| Diseñar | Choose property prefix (`interbank.ads.httpclient.feign.<client>.base-url` / `path...`). |
| Generar | Emit interface, records, config, YAML and test in the user's language. |
| Verificar | No hardcoded URLs/secrets; headers propagated; errors mapped to a domain exception. |

### BMAD — Build phase detail

1. Understand: external contract, service, and headers.
2. Design: record shapes, property keys, error mapping.
3. Build: interface + records + config + YAML + test.
4. Validate: check against the Evaluation checklist.

## Mandatory patterns

Load `references/feign-client-pattern.md` before emitting code.

1. `@FeignClient(name = "...", url = "${interbank.ads.httpclient.feign.<client>.base-url}", configuration = FeignConfig.class)`.
2. Paths are configurable, never hardcoded: `@PostMapping("${interbank.ads.httpclient.feign.<client>.path}")`.
3. Request/response payloads are Java records, not Lombok DTOs.
4. `FeignConfig` is a `@Configuration` exposing a `RequestInterceptor` bean that copies
   `Authorization`, `X-Trace-Id`, `X-Span-Id` and `traceparent` from the current request.
5. Errors are mapped by a custom `ErrorDecoder` to a domain exception (`ExternalServiceException`),
   never leaked as raw `FeignException`.
6. Logged upstream error bodies are sanitized (remove `token`, `password`, `pan`, `cvv`, `pin`).
7. Do not swallow `ErrorDecoder`/interceptor failures silently.
8. Register clients with `@EnableFeignClients(basePackages = "...")` in the `-app` module.

## Clarification questions (ask at most 3)

1. Which BACC service and external API are you integrating?
2. Do you need to add a new client, fix error handling, or propagate headers?
3. What are the success and error status codes you must handle?

## Examples

**Example 1** — new client

Prompt: _¿Cómo creo un cliente Feign en party-lifecycle-management-service para consultar clientes?_

Response:

- Show `@FeignClient(name = "customerClient", url = "${interbank.ads.httpclient.feign.customer.base-url}", configuration = FeignConfig.class)`.
- Show a record request/response and a `@PostMapping`/`@GetMapping` with a configurable path.
- Show the YAML `interbank.ads.httpclient.feign.customer.*` entries.
- Add a Mockito test mocking the client interface.

**Example 2** — error handling

Prompt: _Los errores 4xx/5xx de un servicio externo se muestran como FeignException genérica._

Response:

- Show a custom `ErrorDecoder` mapping status → `ExternalServiceException`.
- Show registration as a `@Bean` in `FeignConfig`.
- Warn to sanitize the body before logging.

**Example 3** — header propagation

Prompt: _El servicio externo no recibe el Authorization ni el trace id en las llamadas Feign._

Response:

- Show a `RequestInterceptor` copying `Authorization`/`X-Trace-Id`/`X-Span-Id`/`traceparent`.
- Note the guard that avoids overwriting an existing `Authorization` header.

## Evaluation

The skill output is valid when:

- [ ] `@FeignClient` uses a configurable `url` and `configuration = FeignConfig.class`.
- [ ] Paths are configurable properties, not hardcoded strings.
- [ ] Payloads are Java records.
- [ ] A `RequestInterceptor` propagates auth/trace headers.
- [ ] A custom `ErrorDecoder` maps failures to a domain exception.
- [ ] No secrets/URLs are hardcoded; they come from config/Key Vault.
- [ ] Logged bodies are sanitized of sensitive fields.
- [ ] A Mockito test snippet for the client is included.

## When not to use this skill

1. The task is Azure Service Bus messaging (use `bcv-azure-service-bus`).
2. The task is designing a REST API contract (use `bcv-openapi-design`).
3. The task is a pure domain/service change with no HTTP client involved.

## Reference loading policy

Load only what is needed:

1. `references/feign-client-pattern.md` — interface + records + FeignConfig + enable wiring + YAML.
2. `references/feign-error-decoder.md` — error decoder patterns and PII sanitization.
