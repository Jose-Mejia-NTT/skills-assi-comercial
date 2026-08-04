---
name: bcv-commons-observability
description: |
  Use this skill whenever the user asks about BCV observability: traces, metrics, Teams alerts, data masking,
  or the use of `@ObservableController`, `@ObservableService` and `@ObservableOperation` in any BCV Java/Spring Boot project.
  Triggers: "observabilidad BCV", "@ObservableController", "@ObservableService", "@ObservableOperation",
  "Teams alert", "data masking", "bcv-observability-token", "Application Insights", "trazas", "métricas",
  "alerta de Teams", "enmascarar datos", "PII logs", "trace id", "OpenTelemetry", "Micrometer",
  "webhook Teams", "commons.exception.teams.webhook.endpoint", or any incident involving missing traces or alerts.
  Applies to: bcv-disb-business-service, bcv-h2h-document-management-service, bcv-h2h-expedient-management-service, bcv-h2h-integration-service.
  Do NOT use for: generic observability tutorials unrelated to BCV, or infrastructure-level AKS monitoring.
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "backend"
  frameworks: ["Spec-Driven Development", "BMAD"]
---

# bcv-commons-observability

## Language rules

- Internal processing, generated code and structural reasoning: **English**.
- Response to the user: **the language of the user's initial message** (default: Spanish).
- Preserve technical terms, class names, property keys and package names in their original form.

## Objective

Guide BCV developers to use the `bcv-commons-observability` library correctly:
annotating components, configuring Teams alerts and data masking, reading traces, and avoiding common mistakes such as hardcoding webhook URLs or swallowing exceptions silently.

## Scope

- `@ObservableController` / `@ObservableService` / `@ObservableOperation`
- Teams alerting via webhook stored in Azure Key Vault
- Data masking for PII in logs
- Trace context propagation (synchronous and WebFlux)
- Dependency and configuration setup
- Troubleshooting missing traces or alerts

## Inputs

Natural language request such as:

- "¿Cómo agrego @ObservableService a un nuevo servicio?"
- "Las alertas de Teams no llegan para errores de expedientes."
- "¿Cómo habilito el data masking para DNI/RUC en los logs?"
- "My controller method is not generating traces in Application Insights."

## Expected output

1. **Project variant identified** — layered (`bcv-disb-business-service`) or hexagonal/clean (`bcv-h2h-*`).
2. **Dependency confirmation** — `bcv-commons-observability` in the executable/app module.
3. **Annotation guidance** — where to place each annotation.
4. **Configuration snippet(s)** — `application.yml`, `bootstrap.yml`, `application-local.yml`.
5. **Security reminder** — webhook URL must come from Key Vault.
6. **Troubleshooting checklist** — token, webhook, data-masking property, log level, trace propagation.

## Workflow

### SDD — Spec Driven Development

| Phase                   | Action                                                                                                                      |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Especificar**         | Parse the request: component to annotate, alert scenario, masking requirement or missing trace.                             |
| **Validar**             | Confirm the project structure and existing observability usage. Ask up to 3 questions if needed.                            |
| **Diseñar**             | Choose annotations and property keys that match the project's conventions.                                                  |
| **Responder / Generar** | Produce the answer, code and config in the user's language.                                                                 |
| **Verificar**           | Check that no secrets are hardcoded and that every `@ObservableService` has `@ObservableOperation` on the relevant methods. |

### BMAD — Build phase detail

1. **Understand** — identify the component type (controller, service, repository, subscriber) and the symptom.
2. **Design** — decide which annotations and properties are needed.
3. **Build** — emit the smallest complete change (annotation, config, test).
4. **Validate** — verify against the checklist in the Evaluation section.

## Mandatory patterns

Load the reference file that matches the user's need before emitting code:

| Topic                                     | Reference file                 |
| ----------------------------------------- | ------------------------------ |
| Dependency and annotation placement       | `references/annotations.md`    |
| Teams alerting configuration and security | `references/teams-alerts.md`   |
| Data masking for PII                      | `references/data-masking.md`   |
| Trace propagation in WebFlux              | `references/webflux-traces.md` |

Core placement rule: `@ObservableController` on controllers, `@ObservableService` on service/repository/subscriber classes, and `@ObservableOperation` on the public methods that should generate spans.

## Clarification questions (ask at most 3)

1. Which BCV project and module are you working on?
2. Are you adding observability to a new component or troubleshooting missing traces/alerts?
3. Do you need Teams alerting, data masking, or both?

## Examples

**Example 1**

Prompt: _¿Cómo agrego @ObservableService a un nuevo servicio en bcv-disb-business-service?_

Response:

- Show the dependency in the `app` POM.
- Show `@ObservableService` on the class and `@ObservableOperation` on each public method.
- Remind that the implementation class (not the interface) should be annotated.
- Provide a unit-test snippet verifying that the annotation is present.

**Example 2**

Prompt: _Las alertas de Teams no llegan cuando falla un servicio en producción._

Response:

- Checklist: `bcv.observability.alerting.enabled: true`, `bcv-webhook-url-teams` resolved from Key Vault, webhook URL valid, network access from AKS to Teams, exception is not swallowed.
- Show how to read the webhook from Key Vault (`bootstrap.yml`).
- Warn against hardcoded `commons.exception.teams.webhook.endpoint`.
- Show how to test the webhook with `curl`.

**Example 3**

Prompt: _How do I enable data masking for DNI/RUC in logs?_

Response:

- Set `bcv.observability.data-masking.enabled: true` in `application.yml`.
- Explain that the library masks fields configured in its own rules; if custom fields are needed, ask the observability team for a rule update.
- Show a log line before/after masking.
- Remind not to log PII as plain text in custom log messages.

## Evaluation

The skill output is valid when:

- [ ] The correct BCV project variant is identified.
- [ ] `bcv-commons-observability` is placed in the executable/app module.
- [ ] `@ObservableController` is used on controllers, `@ObservableService` on services/repositories/subscribers, and `@ObservableOperation` on public methods.
- [ ] The Teams webhook URL is read from Key Vault / environment, never hardcoded.
- [ ] `bootstrap.yml` references `bcv-observability-token` and `bcv-webhook-url-teams` in `secret-keys`.
- [ ] Data masking is enabled via `bcv.observability.data-masking.enabled: true`.
- [ ] WebFlux/reactive code preserves trace context.
- [ ] A troubleshooting checklist is included for incident-style prompts.

## Troubleshooting Notes

Use this section for incident-style prompts. Include the relevant checklist in every response.

### Missing traces in Application Insights

| Symptom                                     | Cause                                                 | Fix                                                                            |
| ------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------ |
| No spans for a controller                   | Missing `@ObservableController`                       | Add annotation to the REST controller class                                    |
| No spans for a service method               | Missing `@ObservableService` / `@ObservableOperation` | Add `@ObservableService` on class and `@ObservableOperation` on public methods |
| Annotation on interface instead of impl     | Only class annotations are scanned                    | Move `@ObservableService` to the implementation class                          |
| Traces stop after `publishOn`/`subscribeOn` | Reactor `Context` not propagated                      | Use `contextWrite` or BCV reactive utilities                                   |

### Teams alerts not arriving

| Symptom                     | Cause                                       | Fix                                                                            |
| --------------------------- | ------------------------------------------- | ------------------------------------------------------------------------------ |
| No alert card in Teams      | `bcv.observability.alerting.enabled: false` | Set to `true` in `application.yml`                                             |
| `404` in alert log          | Invalid webhook URL                         | Verify `bcv-webhook-url-teams` in Key Vault, test with `curl`                  |
| `401` from Teams            | Webhook secret rotated                      | Update secret in Key Vault and restart pods                                    |
| Alert not triggered         | Exception swallowed                         | Ensure exceptions reach the global exception handler or `@ObservableOperation` |
| Missing alert for a service | Service not annotated                       | Add `@ObservableService` and `@ObservableOperation`                            |

### Data masking not working

| Symptom                   | Cause                                           | Fix                                                  |
| ------------------------- | ----------------------------------------------- | ---------------------------------------------------- |
| PII still visible in logs | `bcv.observability.data-masking.enabled: false` | Set to `true`                                        |
| Custom field not masked   | Rule not in library                             | Request a rule update in `bcv-commons-observability` |
| PII in custom message     | Message bypasses masking                        | Rewrite log message to avoid raw PII                 |

### Enable debug logs

```yaml
logging:
  level:
    pe.interbank.bcv.observability: DEBUG
    io.opentelemetry: DEBUG
    com.azure.spring.cloud: DEBUG
```

### Common errors

**`ObservabilityContextNotFoundException`**

- Cause: Trace context requested outside an observable operation.
- Fix: Ensure `@ObservableOperation` is on the entry method or propagate context manually.

**`Webhook URL not found`**

- Cause: `bcv-webhook-url-teams` not in `bootstrap.yml` `secret-keys`.
- Fix: Add the secret key and verify Key Vault access.

**`Token not found`**

- Cause: `bcv-observability-token` missing or not resolved.
- Fix: Add to `secret-keys` and verify Key Vault permissions.

---

## Reference files in BCV repositories

- `bcv-disb-business-service/pom.xml` — `bcv-commons-observability` dependency management.
- `bcv-disb-business-service/bcv-disb-business-service-app/pom.xml`
- `bcv-disb-business-service/bcv-disb-business-service-app/src/main/resources/application.yml`
- `bcv-disb-business-service/bcv-disb-business-service-app/src/main/resources/bootstrap.yml`
- `bcv-disb-business-service/bcv-disb-business-service-app/src/main/java/.../controller/BcvPaymentPromiseController.java`
- `bcv-disb-business-service/bcv-disb-business-service-app/src/main/java/.../service/impl/BcvPaymentPromiseServiceImpl.java`
- `bcv-disb-business-service/bcv-disb-business-service-core/src/main/java/.../repository/BcvCommentRepository.java`
- `bcv-h2h-integration-service/application/src/main/resources/application-local.yml`
- `bcv-h2h-integration-service/infrastructure/entry-points/event-bus-subscriber/.../IntegrationStartProcessSubscriber.java`

---

## Reference Documents

Load from `references/` based on the user's need:

| Reference                      | Content                                                                                                  | When to Load                            |
| ------------------------------ | -------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| `references/annotations.md`    | Dependency, `@ObservableController`, `@ObservableService`, `@ObservableOperation` placement and examples | Adding observability to a new component |
| `references/teams-alerts.md`   | `application.yml`, `bootstrap.yml`, webhook security, manual test                                        | Teams alerting questions                |
| `references/data-masking.md`   | Masking property, rules of thumb, before/after example                                                   | Data masking / PII questions            |
| `references/webflux-traces.md` | Reactor `Context` propagation and common pitfall                                                         | Reactive / WebFlux trace questions      |
