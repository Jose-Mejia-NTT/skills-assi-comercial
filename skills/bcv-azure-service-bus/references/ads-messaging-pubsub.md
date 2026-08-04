# ADS messaging pub/sub pattern

Used in `bcv-h2h-expedient-management-service`.

## Publisher

```java
package pe.interbank.bcv.h2hexpedient.out.broker.publisher;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import pe.interbank.ads.commons.msg.core.message.Message;
import pe.interbank.ads.commons.msg.core.message.MessagePublisher;
import pe.interbank.ads.spring.msg.autoconfigure.message.MessagePublisherRegistry;
import pe.interbank.bcv.h2hexpedient.out.broker.dto.TrackingDto;

import java.time.Duration;

@Slf4j
@Component
public class ExpedientTrackingPublisher {
    private final MessagePublisher messagePublisher;

    @Value("${interbank.ads.messaging.topic.publishers.expedientTrackingPublisher.label}")
    private String trackingPublisherLabel;

    public ExpedientTrackingPublisher(MessagePublisherRegistry registry) {
        this.messagePublisher = registry.getPublisher("expedientTrackingPublisher");
    }

    public void publishMessage(TrackingDto requestDto) {
        Message<TrackingDto> message = new Message.Builder<TrackingDto>()
                .data(requestDto).subject(trackingPublisherLabel)
                .scheduledTime(Duration.ofSeconds(20)).build();
        try {
            messagePublisher.publish(message);
        } catch (Exception e) {
            log.error("[EXPEDIENT_TRACKING_PUBLISHER] Error publishing message: {}", e.getMessage(), e);
        }
    }
}
```

## Subscriber

```java
package pe.interbank.bcv.h2hexpedient.app.in.broker.subscriber;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import pe.interbank.ads.commons.msg.core.message.MessageListener;
import pe.interbank.bcv.h2hexpedient.app.in.broker.dto.ExpedientLoadEventDto;
import pe.interbank.bcv.h2hexpedient.core.command.ExpedientCreationCommandService;

@Slf4j
@RequiredArgsConstructor
public class ExpedientLoadSubscriberHandler implements MessageListener<ExpedientLoadEventDto> {

    private final ExpedientCreationCommandService expedientCreationCommandService;

    @Override
    public void onMessage(ExpedientLoadEventDto expedientLoadEventDto) {
        log.info("Expedient Load Subscriber: {}", expedientLoadEventDto.getPayrollId());
        expedientCreationCommandService.createExpedients(expedientLoadEventDto);
    }
}
```

## Subscriber registration

```java
package pe.interbank.bcv.h2hexpedient.app.in.config;

import org.springframework.context.annotation.Configuration;
import pe.interbank.ads.spring.msg.autoconfigure.message.MessageSubscriberRegistry;
import pe.interbank.ads.spring.msg.autoconfigure.message.MessageSubscriptionConfigurationSupport;
import pe.interbank.bcv.h2hexpedient.app.in.broker.subscriber.*;
import pe.interbank.bcv.h2hexpedient.core.command.*;

@Configuration
public class SubscriberMessagingConfig implements MessageSubscriptionConfigurationSupport {
    private final ExpedientCreationCommandService creationService;
    private final ExpedientCommandService expedientService;

    public SubscriberMessagingConfig(ExpedientCreationCommandService creationService,
                                     ExpedientCommandService expedientService) {
        this.creationService = creationService;
        this.expedientService = expedientService;
    }

    @Override
    public void registerSubscriptionListeners(MessageSubscriberRegistry registry) {
        registry.addListenerForSubscriber("expedientLoadSubscriber",
                new ExpedientLoadSubscriberHandler(creationService));
        registry.addListenerForSubscriber("expedientTrackingSubscriber",
                new TrackingExpedientSubscriberHandler(expedientService));
    }
}
```

## Properties

```yaml
interbank:
  ads:
    messaging:
      platform: AZURE
      connection-string: ${interbank.ads.bcv-secrets.serviceBus.connectionString}
      topic:
        publishers:
          expedientTrackingPublisher:
            connectionString: ${interbank.ads.messaging.connection-string}
            name: h2h.tracking
            label: tracking.expedient.bdj
        subscribers:
          expedientLoadSubscriber:
            connectionString: ${interbank.ads.messaging.connection-string}
            topic-name: h2h.business
            subscription-name: business.expedient-load
```
