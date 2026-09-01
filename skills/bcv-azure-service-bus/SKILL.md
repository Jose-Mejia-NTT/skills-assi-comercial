---
name: bcv-azure-service-bus
description: |
  Use this skill for any BCV Java/Spring Boot question about Azure Service Bus messaging: publishers, subscribers,
  topics, queues, labels, subscriptions, DLQ, retry, lost messages, connection strings, maxConcurrentCalls and the
  ads-spring-boot-starter-messaging (ADS messaging) library.
  Applies to the BACC ecosystem (bcv-bacc-*): topic, messages/queue and Managed Identity variants.
  Do NOT use for generic Azure SDK tutorials or picking a messaging technology (use architecture skills).
metadata:
  version: "1.0.0"
  author: "BCV Architecture Team"
  language: "en"
  category: "backend"
  frameworks: ["Spec-Driven Development", "BMAD"]
---

# bcv-azure-service-bus

## Language handling and output policy

See [references/language-policy.md](references/language-policy.md).

## Objective

Answer BACC-specific Azure Service Bus questions with concrete, project-aligned code and configuration.
The skill must detect which BACC service and configuration sub-variant the user is working on and apply
the exact library and package conventions already used there.

## Project variants

All BACC services (`bcv-bacc-*`) use `ads-spring-boot-starter-messaging`
(`MessagePublisherRegistry` / `MessageSubscriberRegistry`). They differ in transport and property prefix:

| Sub-variant              | Services                             | Transport     | Property prefix                                              |
| ------------------------ | ------------------------------------ | ------------- | ------------------------------------------------------------ |
| Topic + queue            | `account-opening-reporting-service`  | topic + queue | `topic.publishers`, `topic.subscribers`, `queue.subscribers` |
| Messages / queue         | `party-lifecycle-management-service` | queue         | `messages.publishers.<name>.queue`                           |
| Managed Identity (cross) | `channel-activity-service`           | queue + topic | `messagesCross` (`nameSpace` / `manageIdentity`)             |

Typical packages: `out.broker.publisher.*`, `in.broker.subscriber.*`, `in.broker.config.SubscriberMessagingConfig`.

> `service-point-service` (legacy, flat) keeps publishers/subscribers under
> `pe.interbank.bcv.baccservicepoint.publisher` / `pe.interbank.bcv.baccservicepoint.subscriber`.
> See `references/bacc-ads-messaging.md` before emitting code.

> If the user does not name a service, infer it from package names, property prefixes or module names
> found in the current workspace. If still unclear, ask one clarifying question.

## Inputs

Natural language request such as:

- "¿Cómo agrego un publisher para notificar eventos BCV en account-opening-reporting-service?"
- "Mi subscriber en channel-activity-service no consume mensajes de la cola SPL."
- "Add a new queue subscriber for Teradata PN reports in account-opening-reporting-service."
- "Debug why a powers-validation message is not reaching the SPL queue."

## Expected output

1. **Variant identified** — the BACC service and its sub-variant (topic / messages / messagesCross).
2. **Dependency / module changes** — POM snippet or confirmation that the dependency already exists.
3. **Code snippet(s)** — following the exact BACC conventions for that sub-variant.
4. **Configuration snippet(s)** — `bootstrap.yml` or `application.yml` entries.
5. **Test snippet** — Mockito / `@SpringBootTest` pattern used in the same project.
6. **Troubleshooting checklist** — connection string, label/queue match, subscription existence, DLQ, logs.

## Workflow

### SDD — Spec Driven Development

| Phase                   | Action                                                                                                                     |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **Especificar**         | Parse the request: service, publisher/subscriber/both, topic/label/queue/subscription names, payload type, error handling. |
| **Validar**             | Confirm the service and its sub-variant from the workspace. Ask up to 3 questions if critical data is missing.             |
| **Diseñar**             | Choose the correct property prefix, bean naming convention and transport.                                                  |
| **Responder / Generar** | Produce the answer, code and config in the user's language.                                                                |
| **Verificar**           | Check that no secrets are hardcoded, labels/queues match, and errors are logged.                                           |

### BMAD — Build phase detail

1. **Understand** — identify service, role (publisher/subscriber) and payload.
2. **Design** — map topic/label/queue/subscription names to the service's existing property keys.
3. **Build** — emit the smallest complete change (interface, implementation, config, test).
4. **Validate** — verify against the checklist in the Evaluation section.

## Mandatory patterns

Load `references/bacc-ads-messaging.md` and identify the correct sub-variant (topic / messages / messagesCross)
before emitting code. Include only the snippet relevant to the user's request (publisher, subscriber or config)
and replace the example topic/label/queue/subscription names with the real ones from the project.

## Cross-variant rules

1. **Never hardcode secrets** — connection strings, Shared Access Keys and webhook URLs come from Azure Key Vault / Spring Cloud Config.
2. **Always log publish/subscribe errors** — do not swallow exceptions silently.
3. **Use `@ObservableService` + `@ObservableOperation`** on publisher and subscriber handler classes/methods.
4. **Keep labels/queue names consistent** between application config and Service Bus topic/subscription/queue configuration.
5. **Prefer constructor injection** with `@Qualifier` when multiple `MessagePublisher` beans exist.

## Clarification questions (ask at most 3)

1. Which BACC service are you working on?
2. Do you need a publisher, a subscriber, or both?
3. What are the topic/label/queue and subscription names?

## Examples

**Example 1**

Prompt: _¿Cómo agrego un nuevo publisher en account-opening-reporting-service para publicar a un topic?_

Response:

- Identify the `topic.publishers` sub-variant.
- Show the class under `out.broker.publisher` with `@ObservableService`.
- Use `MessagePublisherRegistry.getPublisher("<name>")` and `Message.Builder(...).subject(label)`.
- Show the `@Value("${interbank.ads.messaging.topic.publishers.<name>.label}")` field.
- Provide the YAML snippet and the Key Vault secret reference.
- Add a Mockito unit test snippet mocking `MessagePublisher`.

**Example 2**

Prompt: _Mi subscriber de account-opening-reporting-service no consume mensajes de la cola de reportes Teradata PN._

Response:

- Checklist: verify the queue exists, `queue-name` matches `application.yml`, connection string resolved, no silent exception swallowing.
- Confirm the subscriber is registered in `SubscriberMessagingConfig`.
- Explain DLQ inspection steps.

**Example 3**

Prompt: _Add a new queue publisher in party-lifecycle-management-service using the `messages` prefix._

Response:

- Identify the `messages.publishers` sub-variant.
- Use `@Value("${interbank.ads.messaging.messages.publishers.<name>.queue}")` for the queue name.
- Build the message with `.subject(queue)`.
- Remind the user to create/verify the queue in Azure Service Bus.

## Evaluation

The skill output is valid when:

- [ ] The correct BACC service and sub-variant (topic / messages / messagesCross) are identified.
- [ ] Code snippets follow the existing package and naming conventions.
- [ ] Connection strings are read from Key Vault / environment, never hardcoded.
- [ ] Publisher implementations log errors instead of swallowing them.
- [ ] `@ObservableService` + `@ObservableOperation` are used on publisher/subscriber classes/methods.
- [ ] Configuration snippets use the service's actual property prefixes.
- [ ] A troubleshooting checklist is included for incident-style prompts.

## Troubleshooting Notes

Use this section for incident-style prompts. Include the relevant checklist in every response.

### Publisher not sending messages

| Symptom                                         | Cause                             | Fix                                                                        |
| ----------------------------------------------- | --------------------------------- | -------------------------------------------------------------------------- |
| No logs after publish call                      | Silent exception swallowing       | Wrap publish in `try/catch`, log and re-throw or handle explicitly         |
| `IllegalArgumentException` on connection string | Key Vault secret not resolved     | Verify `bootstrap.yml` `interbank.ads.secrets.serviceBus.connectionString` |
| Message not visible in topic/queue              | Wrong topic/label/queue name      | Confirm topic/label/queue in YAML match Azure Service Bus configuration    |
| `ServiceBusTimeoutException`                    | Network or Service Bus throttling | Check AKS egress, Service Bus SKU limits, retry policy                     |

### Subscriber not consuming messages

| Symptom                       | Cause                        | Fix                                                               |
| ----------------------------- | ---------------------------- | ----------------------------------------------------------------- |
| No messages processed         | Subscription/queue missing   | Create the subscription/queue in Azure or infrastructure pipeline |
| Messages stuck                | Label/queue mismatch         | Ensure `label`/`queue-name` in YAML matches the configured rule   |
| Slow consumption              | `maxConcurrentCalls` too low | Increase to a value appropriate for workload (default often 1)    |
| Messages repeatedly abandoned | Exception in handler         | Log full stack trace, fix business error, check DLQ               |
| Duplicate processing          | No idempotency key           | Add idempotency check based on `messageId` or business key        |

### DLQ inspection

Inspect the dead-letter queue path in Azure Service Bus Explorer (or the equivalent `az servicebus`
CLI commands for topics/subscriptions and queues).

### Enable debug logs

```yaml
logging:
  level:
    com.microsoft.azure.servicebus: DEBUG
    pe.interbank.ads: DEBUG
```

### Common errors

**`MessagingEntityNotFoundException`**

- Cause: Topic, subscription or queue does not exist.
- Fix: Verify Azure resource names and infrastructure deployment.

**`UnauthorizedAccessException`**

- Cause: SAS key expired or lacks `Send`/`Listen` claims (or Managed Identity lacks access).
- Fix: Rotate SAS key in Key Vault or grant the Managed Identity the required role.

**`MessageSizeExceededException`**

- Cause: Payload exceeds Service Bus max message size (256 KB Standard, 1 MB Premium).
- Fix: Compress payload, split message, or move data to storage and send reference.

---

## Reference files in BCV repositories

- `bcv-bacc-account-opening-reporting-service/bcv-bacc-account-opening-reporting-input/pom.xml` — `ads-spring-boot-starter-messaging` + `bcv-commons-observability`.
- `bcv-bacc-account-opening-reporting-service/bcv-bacc-account-opening-reporting-output/.../out/broker/publisher/BvcEventNotificationPublisher.java`
- `bcv-bacc-account-opening-reporting-service/bcv-bacc-account-opening-reporting-input/.../in/broker/config/SubscriberMessagingConfig.java`
- `bcv-bacc-account-opening-reporting-service/bcv-bacc-account-opening-reporting-app/src/main/resources/application.yml` — topic + queue subscribers.
- `bcv-bacc-party-lifecycle-management-service/bcv-bacc-party-lifecycle-management-output/.../out/broker/publisher/PowersValidationPublisher.java` — `messages.publishers.<name>.queue`.
- `bcv-bacc-channel-activity-service/.../src/main/resources/application.yml` — `messagesCross` Managed Identity + `messages` connection-string.

---

## Reference Documents

Load from `references/` based on the identified sub-variant:

| Reference                          | Content                                                                                         | When to Load             |
| ---------------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------ |
| `references/bacc-ads-messaging.md` | BACC `ads-spring-boot-starter-messaging`: topic/messages/queue/MI variants, secrets, real names | any `bcv-bacc-*` service |
