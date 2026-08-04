# bcv-commons-pubsub publisher pattern

Used in `bcv-disb-business-service`.

## Dependency

```xml
<dependency>
    <groupId>pe.interbank.bcv.commons</groupId>
    <artifactId>bcv-commons-pubsub</artifactId>
    <version>${version.library}</version>
</dependency>
```

## Publisher interface

```java
package pe.interbank.bcv.disb.business.service.app.publisher;

public interface OperationSyncPublisher {
    void sendMessageToOperationSync(Long paymentPromiseId);
}
```

## Publisher implementation

```java
package pe.interbank.bcv.disb.business.service.app.publisher.impl;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import pe.interbank.bcv.disb.business.service.app.publisher.OperationSyncPublisher;
import pe.interbank.bcv.disb.business.service.core.util.BeanConstants;
import pe.interbank.bcv.observability.annotation.ObservableOperation;
import pe.interbank.bcv.observability.annotation.ObservableService;
import pe.interbank.commons.pubsub.PublisherClient;
import pe.interbank.commons.utilities.util.JSONUtil;

import java.nio.charset.StandardCharsets;
import java.util.Optional;

@Slf4j
@Component
@ObservableService
public class BcvOperationSyncPublisherImpl implements OperationSyncPublisher {

    @Value("${bcv.disb.prop.operation-sync.operation.pubsub.azure.label}")
    private String operationSyncLabel;

    private final PublisherClient publisherClient;

    public BcvOperationSyncPublisherImpl(@Qualifier(BeanConstants.PUBLISHER_TOPIC_OPERATION_SYNC) PublisherClient publisherClient) {
        this.publisherClient = publisherClient;
    }

    @Override
    @ObservableOperation
    public void sendMessageToOperationSync(Long paymentPromiseId) {
        try {
            String message = JSONUtil.convertToJSONString(new OperationSyncDTO(paymentPromiseId));
            publisherClient.publish(message.getBytes(StandardCharsets.UTF_8), operationSyncLabel, Optional.empty());
            log.info("[OPERATION_SYNC_PUBLISHER] Sent paymentPromiseId={}", paymentPromiseId);
        } catch (Exception e) {
            log.error("[OPERATION_SYNC_PUBLISHER] Failed paymentPromiseId={}", paymentPromiseId, e);
            throw new RuntimeException("Failed to publish operation-sync event", e);
        }
    }

    @lombok.Value
    public static class OperationSyncDTO {
        private final Long paymentPromiseId;
    }
}
```

## PublisherConfig

```java
package pe.interbank.bcv.disb.business.service.app.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.context.WebApplicationContext;
import pe.interbank.bcv.disb.business.service.core.util.BeanConstants;
import pe.interbank.commons.pubsub.PublisherClient;
import pe.interbank.commons.pubsub.impl.AzurePublisherClient;

@Configuration
public class PublisherConfig {
    @Value("${bcv.disb.prop.pubsub.azure.connection}") private String serviceBusConnection;
    @Value("${bcv.disb.prop.pubsub.azure.topic.name}") private String publisherDisbursementSubscription;

    @Bean(BeanConstants.PUBLISHER_TOPIC_OPERATION_SYNC)
    public PublisherClient publisherOperationSyncRequestTopicName(WebApplicationContext ctx) {
        return new AzurePublisherClient(serviceBusConnection, publisherDisbursementSubscription, ctx);
    }
}
```

## Properties

```yaml
bcv:
  disb:
    prop:
      pubsub:
        azure:
          connection: ${bcv-disb-azure-topic-connection}
          topic:
            name: bcv.disbursement
      operation-sync:
        operation:
          pubsub:
            azure:
              label: disbursement.operation-sync
```

The connection string value lives in Azure Key Vault and is referenced in `bootstrap.yml` under `secret-keys`.
