# Trace propagation in WebFlux

In reactive code, trace context is stored in the Reactor `Context`. Switching threads without propagating the context can detach spans from logs and Application Insights.

## Basic pattern

```java
import reactor.core.publisher.Mono;
import reactor.util.context.Context;

public Mono<Result> processReactive(String traceId, Request request) {
    return repository.findById(request.getId())
        .flatMap(entity -> service.transform(entity, request))
        .contextWrite(Context.of("trace-id", traceId))
        .doOnNext(result -> log.info("Processed with trace-id={}", traceId));
}
```

## With BCV observability utilities

If the project provides a `ReactiveTraceContext` utility:

```java
return someMono
    .flatMap(result -> ReactiveTraceContext.withContext(ctx -> {
        log.info("Processing inside trace context");
        return downstreamService.call(result);
    }));
```

## Common pitfall

Switching threads with `publishOn`/`subscribeOn` without propagating the `Context` can detach the span from the log output:

```java
// BAD: context may be lost
return someMono
    .publishOn(Schedulers.boundedElastic())
    .map(this::transform);

// GOOD: preserve context
return someMono
    .publishOn(Schedulers.boundedElastic())
    .contextWrite(Context.of("trace-id", traceId))
    .map(this::transform);
```

## WebFlux controller example

```java
@ObservableController
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    @ObservableOperation
    @GetMapping("/{id}")
    public Mono<OrderDto> getOrder(@PathVariable String id) {
        return orderService.findById(id)
            .doOnNext(order -> log.info("Returning order={}", order.getId()));
    }
}
```

## WebClient trace propagation

```java
return webClient.get()
    .uri(uriBuilder -> uriBuilder.path("/orders/{id}").build(id))
    .headers(headers -> {
        // If a trace context utility is available, use it here
        headers.set("x-trace-id", TraceContext.getTraceId());
    })
    .retrieve()
    .bodyToMono(OrderDto.class);
```

## Verification

Enable debug logging and confirm the same `traceId`/`spanId` appears across logs:

```yaml
logging:
  level:
    pe.interbank.bcv.observability: DEBUG
    io.opentelemetry: DEBUG
```

Expected log output:

```text
2024-08-04T10:00:00.123 [traceId=abc123,spanId=def456] INFO  c.e.o.OrderService - Processing order
2024-08-04T10:00:00.145 [traceId=abc123,spanId=def456] INFO  c.e.o.OrderClient  - Calling downstream
```

If `traceId` changes between logs, the context was lost.
