# BCV observability annotations

## Dependency

`bcv-commons-observability` must be declared in the **executable/app module** (not in core/domain):

```xml
<dependency>
    <groupId>pe.interbank.bcv.commons</groupId>
    <artifactId>bcv-commons-observability</artifactId>
    <version>3.0.0</version>
</dependency>
```

In `bcv-disb-business-service` the dependency is managed in the parent POM and added in the `app` module.

## Placement

| Annotation | Where to put | Why |
|------------|--------------|-----|
| `@ObservableController` | On REST controller classes | Creates a span for the whole HTTP request. |
| `@ObservableService` | On service/repository/subscriber classes | Marks the class as observable. |
| `@ObservableOperation` | On public methods (controller endpoints, service methods, query methods, subscriber handlers) | Creates a span for the operation. |

## Controller example

```java
package pe.interbank.bcv.disb.business.service.app.controller;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import pe.interbank.bcv.observability.annotation.ObservableController;
import pe.interbank.bcv.observability.annotation.ObservableOperation;

@ObservableController
@RestController
@RequestMapping("/disbursement-business/v1/payment-promises")
public class BcvPaymentPromiseController {

    @ObservableOperation
    @Operation(summary = "Create payment promise")
    @ApiResponse(responseCode = "200", description = "Created")
    @PostMapping(consumes = MediaType.APPLICATION_JSON_VALUE, produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<PaymentPromiseResponseDto> createPaymentPromise(
            @RequestBody PaymentPromiseRequestDto requestDto) {
        // delegate to service/facade
    }
}
```

## Service example

```java
package pe.interbank.bcv.disb.business.service.app.service.impl;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import pe.interbank.bcv.disb.business.service.app.service.PaymentPromiseService;
import pe.interbank.bcv.observability.annotation.ObservableOperation;
import pe.interbank.bcv.observability.annotation.ObservableService;

@Slf4j
@Service
@ObservableService
@RequiredArgsConstructor
public class BcvPaymentPromiseServiceImpl implements PaymentPromiseService {

    @ObservableOperation
    public PaymentPromiseDto createPaymentPromise(PaymentPromiseRequestDto requestDto) {
        // business logic
    }
}
```

## Repository example

```java
package pe.interbank.bcv.disb.business.service.core.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import pe.interbank.bcv.disb.business.service.core.domain.entity.Comments;
import pe.interbank.bcv.observability.annotation.ObservableOperation;
import pe.interbank.bcv.observability.annotation.ObservableService;

import java.time.ZonedDateTime;

@Repository
@ObservableService
public interface BcvCommentRepository extends JpaRepository<Comments, Long> {

    @ObservableOperation
    @Modifying
    @Query(value = "UPDATE Disbursement.Comments SET content = :content, updatedOn = :updatedOn WHERE id = :id", nativeQuery = true)
    void updateCommentsById(
        @Param("id") Long id,
        @Param("content") String content,
        @Param("updatedOn") ZonedDateTime updatedOn);
}
```

## WebFlux / subscriber example

```java
package pe.interbank.bcv.h2h_integration.subscriber.listener;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import pe.interbank.bcv.observability.annotation.ObservableOperation;
import pe.interbank.commons.topic.sub.SubscriberMessageActions;
import pe.interbank.commons.topic.sub.client.SubscriberHandler;

import java.util.Map;

@Slf4j
@Service
@RequiredArgsConstructor
public class IntegrationStartProcessSubscriber implements SubscriberHandler {

    @Override
    @ObservableOperation
    public SubscriberMessageActions handler(Map<String, String> map, byte[] bytes) {
        // processing
        return SubscriberMessageActions.COMPLETE;
    }
}
```
