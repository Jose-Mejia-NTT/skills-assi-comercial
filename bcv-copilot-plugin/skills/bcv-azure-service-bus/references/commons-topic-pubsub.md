# bcv-commons-topic pub/sub pattern

Used in `bcv-h2h-integration-service`.

## Publisher client

```java
package pe.interbank.bcv.h2h_integration.publisher.client;

import org.springframework.stereotype.Service;
import org.springframework.web.context.WebApplicationContext;
import pe.interbank.bcv.h2h_integration.publisher.property.IntegrationPublisherProperties;
import pe.interbank.commons.topic.pub.AzurePublisherClient;

@Service
public class IntegrationPublisherClient extends AzurePublisherClient {
    public IntegrationPublisherClient(IntegrationPublisherProperties properties,
                                      WebApplicationContext webApplicationContext) {
        super(properties.getConnection(), properties.getTopic(), webApplicationContext);
    }
}
```

## Publisher properties

```java
package pe.interbank.bcv.h2h_integration.publisher.property;

import lombok.Getter;
import lombok.Setter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Getter
@Setter
@Configuration
@ConfigurationProperties(prefix = "h2h.publisher.integration")
public class IntegrationPublisherProperties extends PublisherProperties {
    private TopicProperties process;
    private TopicProperties notification;
}
```

## Subscriber

```java
package pe.interbank.bcv.h2h_integration.subscriber.listener;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import pe.interbank.bcv.observability.annotation.ObservableOperation;
import pe.interbank.commons.topic.sub.SubscriberMessageActions;
import pe.interbank.commons.topic.sub.client.SubscriberHandler;

import java.nio.charset.StandardCharsets;
import java.util.Map;

@Slf4j
@Service
public class IntegrationStartProcessSubscriber implements SubscriberHandler {
    private static final String PREFIX = "h2h.subscriber.integration";

    @Override
    public String getPropertyBase() { return PREFIX; }

    @Override
    @ObservableOperation
    public SubscriberMessageActions handler(Map<String, String> map, byte[] bytes) {
        String json = new String(bytes, StandardCharsets.UTF_8);
        log.info("Integration Start Process Subscriber: {}", json);
        try {
            // business logic
            return SubscriberMessageActions.COMPLETE;
        } catch (Exception e) {
            log.error("Error processing integration start message", e);
            return SubscriberMessageActions.ABANDON;
        }
    }
}
```

## Properties

```yaml
h2h:
  publisher:
    integration:
      connection: ${bcv-h2h-sb-connection-string}
      topic: h2h.integration
      process:
        label: integration.process
      notification:
        label: integration.notification
  subscriber:
    integration-start:
      pubsub:
        azure:
          maxConcurrentCalls: 2
          subscription: integration.process
          connection: ${bcv-h2h-sb-connection-string}
          label: integration.process
          topic: h2h.integration
```
