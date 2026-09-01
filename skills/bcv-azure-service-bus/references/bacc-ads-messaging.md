# BACC ADS messaging (ads-spring-boot-starter-messaging)

Used by all 7 `bcv-bacc-*` services (account-opening-reporting, channel-activity, compliance,
current-account, customer, party-lifecycle-management, service-point).

BACC uses the `pe.interbank.ads.commons:ads-spring-boot-starter-messaging` library with
BACC-specific package layout, property prefixes and three configuration sub-variants
(topic, messages/queue, and cross-tenant Managed Identity).

## Dependency

- Subscribers live in the `-input` module (entry points).
- Publishers live in the `-output` module (driven adapters).

```xml
<!-- input module (subscribers) -->
<dependency>
    <groupId>pe.interbank.ads.commons</groupId>
    <artifactId>ads-spring-boot-starter-messaging</artifactId>
</dependency>
<dependency>
    <groupId>pe.interbank.bcv.commons</groupId>
    <artifactId>bcv-commons-observability</artifactId>
</dependency>

<!-- output module (publishers) -->
<dependency>
    <groupId>pe.interbank.ads.commons</groupId>
    <artifactId>ads-spring-boot-starter-messaging</artifactId>
</dependency>
```

## Imports

```java
import pe.interbank.ads.commons.msg.core.message.Message;
import pe.interbank.ads.commons.msg.core.message.MessagePublisher;
import pe.interbank.ads.commons.msg.core.message.MessageListener;
import pe.interbank.ads.spring.msg.autoconfigure.message.MessagePublisherRegistry;
import pe.interbank.ads.spring.msg.autoconfigure.message.MessageSubscriberRegistry;
import pe.interbank.ads.spring.msg.autoconfigure.message.MessageSubscriptionConfigurationSupport;
import pe.interbank.bcv.observability.annotation.ObservableOperation;
import pe.interbank.bcv.observability.annotation.ObservableService;
```

## Package conventions

| Concern | Package |
| --- | --- |
| Publisher | `pe.interbank.bcv.<service>.out.broker.publisher` |
| Publisher DTO | `pe.interbank.bcv.<service>.out.broker.dto` |
| Subscriber handler | `pe.interbank.bcv.<service>.in.broker.subscriber` |
| Subscriber DTO | `pe.interbank.bcv.<service>.in.broker.dto` |
| Subscriber registration | `pe.interbank.bcv.<service>.in.broker.config.SubscriberMessagingConfig` |

> `service-point-service` (legacy, non-hexagonal) keeps publishers/subscribers under
> `pe.interbank.bcv.baccservicepoint.publisher` and `pe.interbank.bcv.baccservicepoint.subscriber`.

## Publisher (topic variant — used by account-opening-reporting-service)

```java
package pe.interbank.bcv.baccaccountopeningreporting.out.broker.publisher;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import pe.interbank.ads.commons.msg.core.message.Message;
import pe.interbank.ads.commons.msg.core.message.MessagePublisher;
import pe.interbank.ads.spring.msg.autoconfigure.message.MessagePublisherRegistry;
import pe.interbank.bcv.observability.annotation.ObservableOperation;
import pe.interbank.bcv.observability.annotation.ObservableService;

@Slf4j
@Component
@ObservableService
public class BvcEventNotificationPublisher {
    private final MessagePublisher messagePublisher;

    @Value("${interbank.ads.messaging.topic.publishers.bvcEventNotificationPublisher.label}")
    private String label;

    public BvcEventNotificationPublisher(MessagePublisherRegistry registry) {
        this.messagePublisher = registry.getPublisher("bvcEventNotificationPublisher");
    }

    @ObservableOperation
    public void publishMessage(EventReportOutDto dto, Map<String, String> properties) {
        Message<EventReportOutDto> message = new Message.Builder<EventReportOutDto>()
                .data(dto)
                .subject(label)
                .properties(properties)
                .build();
        try {
            messagePublisher.publish(message);
        } catch (Exception e) {
            log.error("[BVC_EVENT_NOTIFICATION_PUBLISHER] Error publishing message: {}", e.getMessage(), e);
        }
    }
}
```

## Publisher (messages/queue variant — used by party-lifecycle-management-service)

```java
package pe.interbank.bcv.baccpartylifecyclemanagement.out.broker.publisher;

@Slf4j
@Component
@ObservableService
public class PowersValidationPublisher {
    private final MessagePublisher messagePublisher;

    @Value("${interbank.ads.messaging.messages.publishers.powersValidationBcvReqPublisher.queue}")
    private String queue;

    public PowersValidationPublisher(MessagePublisherRegistry registry) {
        this.messagePublisher = registry.getPublisher("powersValidationBcvReqPublisher");
    }

    @ObservableOperation
    public void publishMessage(PowersValidationOutDto dto) {
        Message<PowersValidationOutDto> message = new Message.Builder<PowersValidationOutDto>()
                .data(dto)
                .subject(queue)
                .scheduledTime(Duration.ofSeconds(20))
                .build();
        try {
            messagePublisher.publish(message);
        } catch (Exception e) {
            log.error("[POWERS_VALIDATION_PUBLISHER] Error publishing message: {}", e.getMessage(), e);
        }
    }
}
```

## Subscriber handler

```java
package pe.interbank.bcv.baccaccountopeningreporting.in.broker.subscriber;

import lombok.AllArgsConstructor;
import pe.interbank.ads.commons.msg.core.message.MessageListener;
import pe.interbank.bcv.baccaccountopeningreporting.core.port.in.command.ReportProcessInCommandPort;
import pe.interbank.bcv.observability.annotation.ObservableOperation;
import pe.interbank.bcv.observability.annotation.ObservableService;

@AllArgsConstructor
@ObservableService
public class ReportTeradataNaturalPersonSubscriberHandler implements MessageListener<TeradataNaturalPerson> {

    private final ReportProcessInCommandPort reportProcessInCommandPort;

    @Override
    @ObservableOperation
    public void onMessage(TeradataNaturalPerson payload) {
        reportProcessInCommandPort.processReportTeradataNaturalPerson(
            ReportMapper.MAPPER.toReportTeradataNPDomain(payload));
    }
}
```

## Subscriber registration

```java
package pe.interbank.bcv.baccaccountopeningreporting.in.broker.config;

import lombok.AllArgsConstructor;
import org.springframework.context.annotation.Configuration;
import pe.interbank.ads.spring.msg.autoconfigure.message.MessageSubscriberRegistry;
import pe.interbank.ads.spring.msg.autoconfigure.message.MessageSubscriptionConfigurationSupport;

@AllArgsConstructor
@Configuration
public class SubscriberMessagingConfig implements MessageSubscriptionConfigurationSupport {
    private final ReportProcessInCommandPort reportProcessInCommandPort;
    private final CurrentAccountCommercialExpedientInCommandPort currentAccountCommercialExpedientInCommandPort;

    @Override
    public void registerSubscriptionListeners(MessageSubscriberRegistry registry) {
        registry.addListenerForSubscriber(
                "reportTeradataNPSubscriber",
                new ReportTeradataNaturalPersonSubscriberHandler(reportProcessInCommandPort),
                TeradataNaturalPerson.class);

        registry.addListenerForSubscriber(
                "currentAccountExpedientSyncUpsertSubscriber",
                new CurrentAccountCommercialExpedientSyncSaveSubscriberHandler(currentAccountCommercialExpedientInCommandPort),
                CurrentAccountExpedient.class);
    }
}
```

## Configuration sub-variants

BACC services do NOT all use the same property prefix. Detect which one the target service uses.

### 1. Topic variant (`account-opening-reporting-service`)

```yaml
interbank:
  ads:
    messaging:
      platform: AZURE
      connection-string: ${interbank.ads.secrets.serviceBus.connectionString}
      topic:
        publishers:
          bvcEventNotificationPublisher:
            connectionString: ${interbank.ads.messaging.connection-string}
            name: bcv.event
            label: event.record
        subscribers:
          currentAccountExpedientSyncUpsertSubscriber:
            connectionString: ${interbank.ads.messaging.connection-string}
            topic-name: bcv.bacc.report
            subscription-name: bacc.report.commercial.cce_upsert
      queue:
        subscribers:
          reportTeradataNPSubscriber:
            connectionString: ${interbank.ads.messaging.connection-string}
            queue-name: que-bcv-ars-reporting-data-ingestion-pn-req-01
```

### 2. Messages/queue variant (`party-lifecycle-management-service`)

Uses `messages.publishers` / `messages.subscribers` and the `queue` key holds the queue name
(also used as the message subject/label).

```yaml
interbank:
  ads:
    messaging:
      connection-string: "${ibk-nr_bcv-bacc-sb-connection-string}"
      messages:
        publishers:
          powersValidationBcvReqPublisher:
            connectionString: ${interbank.ads.messaging.connection-string}
            queue: que-bcv-channel-activity-powers-read-req-01
        subscribers:
          powersResponseSubscriber:
            connectionString: ${interbank.ads.messaging.connection-string}
            queue: que-bcv-channel-activity-powers-read-res-01
```

### 3. Cross-tenant Managed Identity variant (`channel-activity-service`)

`channel-activity-service` authenticates to SPL queues using Managed Identity (`messagesCross`)
alongside regular connection-string publishers (`messages`).

```yaml
interbank:
  ads:
    messaging:
      managed-identity-enabled: true
      connection-string: "${ibk-nr_bcv-bacc-sb-connection-string}"
      connection-string-sb-hypl: "${ibk-nr_bcv-bacc-hypl-sb-connection-string}"
      nameSpaceCross: "${ibk-nr_bcv-bacc-sb-nameSpace}"
      manageIdentityCross: "${ibk-nr_bcv-bacc-sb-manageidentity}"
      messagesCross:
        publishers:
          powersValidationMIPublisher:
            nameSpace: ${interbank.ads.messaging.nameSpaceCross}
            manageIdentity: ${interbank.ads.messaging.manageIdentityCross}
            queue: que-spl-partidaregistral-evaluar-spl-req-01
        subscribers:
          powersResponseSubscriberMIHandler:
            nameSpace: ${interbank.ads.messaging.nameSpaceCross}
            manageIdentity: ${interbank.ads.messaging.manageIdentityCross}
            queue: que-spl-partidaregistral-evaluar-spl-res-01
      messages:
        publishers:
          bcvNotificationPublisher:
            platform: AZURE_SERVICE_BUS
            connectionString: ${interbank.ads.messaging.connection-string}
            topic: bcv.notification
```

## Secret naming

Connection strings are never hardcoded. They are resolved through `bootstrap.yml`:

```yaml
interbank:
  ads:
    secrets:
      serviceBus:
        connectionString: ${ibk-nr_bcv-bacc-sb-connection-string}
```

Real secret keys seen in BACC: `ibk-nr_bcv-bacc-sb-connection-string`,
`ibk-nr_bcv-bacc-hypl-sb-connection-string`, `ibk-nr_bcv-bacc-sb-manageidentity`,
`ibk-nr_bcv-bacc-sb-nameSpace`.

## Real queues/topics seen in BACC

| Kind | Name | Used by |
| --- | --- | --- |
| Topic | `bcv.event` | reporting, PLM, service-point |
| Topic | `bcv.bacc.report` | reporting, PLM |
| Topic | `bcv.notification` | channel-activity |
| Topic | `hyperloop.notification` / `hyperloop.document` | channel-activity |
| Queue | `que-bcv-ars-reporting-data-ingestion-pn-req-01` | reporting (PN Teradata) |
| Queue | `que-bcv-ars-reporting-data-ingestion-pj-req-01` | reporting (PJ Teradata) |
| Queue | `que-bcv-ars-reporting-data-ingestion-gtp-req-01` | reporting (GTP) |
| Queue | `que-bcv-channel-activity-powers-read-req-01` / `-res-01` | PLM, channel-activity |
| Queue | `que-spl-partidaregistral-evaluar-spl-req-01` / `-res-01` | channel-activity (SPL, MI) |

## Observability

Every publisher/subscriber uses `@ObservableService` on the class and `@ObservableOperation` on
the publish/onMessage method (see the `bcv-commons-observability` skill).

## Evaluation checklist

- [ ] Correct BACC service and configuration sub-variant identified (topic / messages / messagesCross).
- [ ] Publisher/subscriber placed in `out.broker.*` / `in.broker.*` (or legacy `publisher`/`subscriber` in service-point).
- [ ] `MessagePublisherRegistry.getPublisher(<name>)` name matches the YAML key.
- [ ] `@Value` reads the correct prefix (`topic.publishers.<name>.label` or `messages.publishers.<name>.queue`).
- [ ] No hardcoded connection strings; they come from `interbank.ads.secrets.*`.
- [ ] `@ObservableService` + `@ObservableOperation` present.
- [ ] Errors are logged, not swallowed.
- [ ] Managed Identity variant uses `nameSpace`/`manageIdentity`, not connection strings, for SPL cross-tenant queues.
