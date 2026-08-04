---
name: bcv-azure-service-bus
description: |
  Use this skill for any BCV Java/Spring Boot question about Azure Service Bus messaging: publishers, subscribers,
  topics, labels, subscriptions, DLQ, retry, lost messages, connection strings, maxConcurrentCalls or the libraries
  bcv-commons-pubsub / bcv-commons-topic / ADS messaging. Applies to all BCV projects.
  Do NOT use for generic Azure SDK tutorials or picking a messaging technology (use architecture skills).
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "backend"
  frameworks: ["Spec-Driven Development", "BMAD"]
---

# bcv-azure-service-bus

## Language rules

- Internal processing, generated code and structural reasoning: **English**.
- Response to the user: **the language of the user's initial message** (default: Spanish).
- Preserve technical terms, class names, property keys and package names in their original form.

## Objective

Answer BCV-specific Azure Service Bus questions with concrete, project-aligned code and configuration.
The skill must detect which BCV project variant the user is working on and apply the exact library and package conventions already used there.

## Project variants

| Project                                | Library / abstraction                                                    | Pattern              | Typical module/package                                                                    |
| -------------------------------------- | ------------------------------------------------------------------------ | -------------------- | ----------------------------------------------------------------------------------------- |
| `bcv-disb-business-service`            | `bcv-commons-pubsub` (`PublisherClient` / `AzurePublisherClient`)        | Publisher only       | `app.publisher.*`, `app.config.PublisherConfig`                                           |
| `bcv-h2h-document-management-service`  | `commons.audit` auto-config                                              | Audit publisher only | `application-local.yml` `commons.audit.*`                                                 |
| `bcv-h2h-expedient-management-service` | ADS messaging (`MessagePublisherRegistry` / `MessageSubscriberRegistry`) | Pub/sub              | `out.broker.publisher.*`, `in.broker.subscriber.*`, `in.config.SubscriberMessagingConfig` |
| `bcv-h2h-integration-service`          | `bcv-commons-topic` (`AzurePublisherClient` / `SubscriberHandler`)       | Pub/sub              | `publisher.client.*`, `publisher.property.*`, `subscriber.listener.*`                     |

> If the user does not name a project, infer it from package names, property prefixes or module names found in the current workspace. If still unclear, ask one clarifying question.

## Inputs

Natural language request such as:

- "¿Cómo agrego un publisher para operation-sync en bcv-disb-business-service?"
- "My subscriber in h2h-integration is not reading messages from h2h.integration topic."
- "Add a new label to the expedient tracking publisher."
- "Debug why audit events are not reaching bcv.audit."

## Expected output

1. **Variant identified** — the project and library pattern being used.
2. **Dependency / module changes** — POM snippet or confirmation that the dependency already exists.
3. **Code snippet(s)** — following the exact BCV conventions for that variant.
4. **Configuration snippet(s)** — `application-local.yml`, `bootstrap.yml` or `application.yml` entries.
5. **Test snippet** — Mockito / `@SpringBootTest` pattern used in the same project.
6. **Troubleshooting checklist** — connection string, label match, subscription existence, DLQ, logs.

## Workflow

### SDD — Spec Driven Development

| Phase                   | Action                                                                                                                      |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Especificar**         | Parse the request: project, publisher/subscriber/both, topic/label/subscription names, payload type, error-handling needs.  |
| **Validar**             | Confirm the project variant and existing conventions from the workspace. Ask up to 3 questions if critical data is missing. |
| **Diseñar**             | Choose the correct BCV library pattern, bean naming convention and property prefix.                                         |
| **Responder / Generar** | Produce the answer, code and config in the user's language.                                                                 |
| **Verificar**           | Check that no secrets are hardcoded, labels match the topic, and errors are logged.                                         |

### BMAD — Build phase detail

1. **Understand** — identify project, role (publisher/subscriber) and payload.
2. **Design** — map topic/label/subscription names to the project's existing property keys.
3. **Build** — emit the smallest complete change (interface, implementation, config, test).
4. **Validate** — verify against the checklist in the Evaluation section.

## Mandatory patterns by variant

Load the reference file that matches the identified project before emitting code:

| Project                                | Library                          | Reference file                       |
| -------------------------------------- | -------------------------------- | ------------------------------------ |
| `bcv-disb-business-service`            | `bcv-commons-pubsub` (publisher) | `references/pubsub-publisher.md`     |
| `bcv-h2h-integration-service`          | `bcv-commons-topic` (pub/sub)    | `references/commons-topic-pubsub.md` |
| `bcv-h2h-expedient-management-service` | ADS messaging (pub/sub)          | `references/ads-messaging-pubsub.md` |
| `bcv-h2h-document-management-service`  | `commons.audit` (audit topic)    | `references/commons-audit.md`        |

Only include the snippet that is relevant to the user's request (publisher, subscriber or audit config) and replace the example topic/label/subscription names with the real ones from the project.

## Cross-variant rules

1. **Never hardcode secrets** — connection strings, Shared Access Keys and webhook URLs come from Azure Key Vault / Spring Cloud Config.
2. **Always log publish/subscribe errors** — do not swallow exceptions silently.
3. **Use `@ObservableOperation`** on publisher and subscriber handler methods.
4. **Keep labels consistent** between application config and Service Bus topic/subscription configuration.
5. **Return meaningful actions** in `bcv-commons-topic` subscribers (`COMPLETE`, `ABANDON`, `DEAD_LETTER`).
6. **Prefer constructor injection** with `@Qualifier` when multiple `PublisherClient` beans exist.

## Clarification questions (ask at most 3)

1. Which BCV project are you working on?
2. Do you need a publisher, a subscriber, or both?
3. What are the topic, label and subscription names?

## Examples

**Example 1**

Prompt: _¿Cómo agrego un nuevo publisher de Azure Service Bus en bcv-disb-business-service?_

Response:

- Identify the `bcv-commons-pubsub` variant.
- Show the interface + implementation under `app.publisher`.
- Show the `@Bean` registration in `PublisherConfig`.
- Provide the YAML property snippet and the Key Vault secret reference.
- Add a Mockito unit test snippet mocking `PublisherClient`.

**Example 2**

Prompt: _My subscriber in bcv-h2h-integration-service is not consuming messages from h2h.integration._

Response:

- Checklist: verify subscription exists, label matches `application-local.yml`, `maxConcurrentCalls` > 0, connection string resolved, no silent exception swallowing.
- Show how to enable debug logs for `com.microsoft.azure.servicebus`.
- Explain DLQ inspection steps.

**Example 3**

Prompt: _Add a new label `disbursement.notification.bdj` to the disbursement publisher in h2h-integration._

Response:

- Update `DisbursementPublisherProperties` with a new `TopicProperties notificationBdj`.
- Update `application-local.yml` under `h2h.publisher.disbursement`.
- Update the adapter that uses the client.
- Remind the user to create/verify the label in the topic.

## Evaluation

The skill output is valid when:

- [ ] The correct BCV project variant and library are identified.
- [ ] Code snippets follow the existing project package and naming conventions.
- [ ] Connection strings are read from Key Vault / environment, never hardcoded.
- [ ] Publisher implementations log errors instead of swallowing them.
- [ ] Subscriber handlers return the correct action (`COMPLETE`, `ABANDON`, `DEAD_LETTER`).
- [ ] `@ObservableOperation` is used on publisher/subscriber methods.
- [ ] Configuration snippets use the project's actual property prefixes.
- [ ] A troubleshooting checklist is included for incident-style prompts.

## Troubleshooting Notes

Use this section for incident-style prompts. Include the relevant checklist in every response.

### Publisher not sending messages

| Symptom                                         | Cause                             | Fix                                                                            |
| ----------------------------------------------- | --------------------------------- | ------------------------------------------------------------------------------ |
| No logs after publish call                      | Silent exception swallowing       | Wrap publish in `try/catch`, log and re-throw or handle explicitly             |
| `IllegalArgumentException` on connection string | Key Vault secret not resolved     | Verify `bootstrap.yml` `secret-keys` and `bcv-*-connection` value in Key Vault |
| Message not visible in topic                    | Wrong topic/label name            | Confirm topic and label in YAML match Azure Service Bus configuration          |
| `ServiceBusTimeoutException`                    | Network or Service Bus throttling | Check AKS egress, Service Bus SKU limits, retry policy                         |

### Subscriber not consuming messages

| Symptom                       | Cause                        | Fix                                                            |
| ----------------------------- | ---------------------------- | -------------------------------------------------------------- |
| No messages processed         | Subscription does not exist  | Create subscription in Azure Portal or infrastructure pipeline |
| Messages stuck in topic       | Label mismatch               | Ensure `label` in YAML matches subscription rule label filter  |
| Slow consumption              | `maxConcurrentCalls` too low | Increase to a value appropriate for workload (default often 1) |
| Messages repeatedly abandoned | Exception in handler         | Log full stack trace, fix business error, check DLQ            |
| Duplicate processing          | No idempotency key           | Add idempotency check based on `messageId` or business key     |

### DLQ inspection

For `bcv-commons-topic`:

```bash
# Peek dead-letter messages for a subscription
az servicebus topic subscription show-dead-letter-messages-details \
  --resource-group <rg> \
  --namespace-name <ns> \
  --topic-name <topic> \
  --subscription-name <subscription> \
  --output table
```

For ADS messaging, inspect the dead-letter queue path in Azure Service Bus Explorer.

### Enable debug logs

```yaml
logging:
  level:
    com.microsoft.azure.servicebus: DEBUG
    pe.interbank.commons.pubsub: DEBUG
    pe.interbank.commons.topic: DEBUG
    pe.interbank.ads: DEBUG
```

### Common errors

**`MessagingEntityNotFoundException`**

- Cause: Topic or subscription does not exist.
- Fix: Verify Azure resource names and infrastructure deployment.

**`UnauthorizedAccessException`**

- Cause: SAS key expired or lacks `Send`/`Listen` claims.
- Fix: Rotate SAS key in Key Vault and update `secret-keys`.

**`MessageSizeExceededException`**

- Cause: Payload exceeds Service Bus max message size (256 KB Standard, 1 MB Premium).
- Fix: Compress payload, split message, or move data to storage and send reference.

---

## Reference files in BCV repositories

- `bcv-disb-business-service/pom.xml` — `bcv-commons-pubsub` dependency.
- `bcv-disb-business-service/bcv-disb-business-service-app/src/main/java/.../config/PublisherConfig.java`
- `bcv-disb-business-service/bcv-disb-business-service-app/src/main/java/.../publisher/impl/BcvOperationSyncPublisherImpl.java`
- `bcv-disb-business-service/bcv-disb-business-service-app/src/main/resources/bootstrap.yml`
- `bcv-h2h-integration-service/application/src/main/resources/application-local.yml`
- `bcv-h2h-integration-service/infrastructure/driven-adapters/event-bus-publisher/...`
- `bcv-h2h-integration-service/infrastructure/entry-points/event-bus-subscriber/.../IntegrationStartProcessSubscriber.java`
- `bcv-h2h-expedient-management-service/bcv-h2h-expedient-management-app/src/main/resources/application-local.yml`
- `bcv-h2h-expedient-management-service/bcv-h2h-expedient-management-output/.../broker/publisher/ExpedientTrackingPublisher.java`
- `bcv-h2h-expedient-management-service/bcv-h2h-expedient-management-input/.../broker/subscriber/ExpedientLoadSubscriberHandler.java`
- `bcv-h2h-document-management-service/bcv-h2h-document-management-app/src/main/resources/application-local.yml`

---

## Reference Documents

Load from `references/` based on the identified project variant:

| Reference                            | Content                                                                | When to Load                                   |
| ------------------------------------ | ---------------------------------------------------------------------- | ---------------------------------------------- |
| `references/pubsub-publisher.md`     | `bcv-commons-pubsub` publisher interface, implementation, config, YAML | `bcv-disb-business-service` publisher          |
| `references/commons-topic-pubsub.md` | `bcv-commons-topic` publisher client, properties, subscriber, YAML     | `bcv-h2h-integration-service` pub/sub          |
| `references/ads-messaging-pubsub.md` | ADS messaging publisher, subscriber, registration, YAML                | `bcv-h2h-expedient-management-service` pub/sub |
| `references/commons-audit.md`        | `commons.audit` auto-configuration for audit events                    | `bcv-h2h-document-management-service` audit    |
