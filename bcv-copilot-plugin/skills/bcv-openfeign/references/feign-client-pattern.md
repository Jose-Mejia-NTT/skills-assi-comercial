# Feign client pattern (BACC)

Use this reference to create a BACC OpenFeign client following the ecosystem conventions.

## Dependency

The `spring-cloud-starter-openfeign` dependency lives in the `-output` module (clients) and the
`-app` module (wiring). Do not add it to `-core`.

## Client interface

```java
package pe.interbank.bcv.baccpartylifecyclemanagement.out.client;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.*;
import pe.interbank.bcv.baccpartylifecyclemanagement.out.client.config.FeignConfig;
import pe.interbank.bcv.baccpartylifecyclemanagement.out.client.record.*;

import java.util.List;

@FeignClient(
        name = "customerClient",
        url = "${interbank.ads.httpclient.feign.customer.base-url}",
        configuration = FeignConfig.class
)
public interface BaccCustomerClient {

    @PostMapping(path = "${interbank.ads.httpclient.feign.customer.path}")
    CustomerResponseRecord createCustomer(ClientRequestRecord request);

    @GetMapping(path = "${interbank.ads.httpclient.feign.customer.pathQueryLegalEntity}")
    CustomerLegalEntityRecord findLegalEntityById(@PathVariable("id") Long id);

    @GetMapping(path = "${interbank.ads.httpclient.feign.customer.legal-representative-path}/query")
    List<LegalRepresentativeResponseRecord> listLegalRepresentatives(
            @RequestParam(value = "documentNumber", required = false) String documentNumber,
            @RequestParam(value = "phone", required = false) String phone);
}
```

Rules:

- `name` is the bean qualifier; keep it unique per service.
- `url` uses `${interbank.ads.httpclient.feign.<client>.base-url}`.
- Every path uses `${interbank.ads.httpclient.feign.<client>.path...}`.
- Request/response types are Java records (no Lombok on these).

## Records

```java
package pe.interbank.bcv.baccpartylifecyclemanagement.out.client.record;

public record ClientRequestRecord(String documentType, String documentNumber) {}

public record CustomerResponseRecord(String id, String status) {}
```

## FeignConfig (interceptor)

```java
package pe.interbank.bcv.baccpartylifecyclemanagement.out.client.config;

import feign.RequestInterceptor;
import feign.RequestTemplate;
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.util.StringUtils;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import java.util.Map;

@Configuration
public class FeignConfig {

    private static final String TRACE_ID_HEADER = "X-Trace-Id";
    private static final String SPAN_ID_HEADER = "X-Span-Id";
    private static final String TRACEPARENT_HEADER = "traceparent";
    private static final String AUTH_HEADER = "Authorization";

    @Bean
    public RequestInterceptor requestInterceptor() {
        return (RequestTemplate template) -> {
            RequestAttributes attrs = RequestContextHolder.getRequestAttributes();
            if (attrs instanceof ServletRequestAttributes servletAttrs) {
                HttpServletRequest request = servletAttrs.getRequest();
                Map.of(
                        AUTH_HEADER, request.getHeader(AUTH_HEADER),
                        TRACE_ID_HEADER, request.getHeader(TRACE_ID_HEADER),
                        SPAN_ID_HEADER, request.getHeader(SPAN_ID_HEADER),
                        TRACEPARENT_HEADER, request.getHeader(TRACEPARENT_HEADER)
                ).forEach((key, value) -> {
                    if (StringUtils.hasText(value)
                            && !(AUTH_HEADER.equals(key) && template.headers().containsKey(AUTH_HEADER))) {
                        template.header(key, value);
                    }
                });
            }
        };
    }
}
```

Notes:

- Never overwrite an existing `Authorization` header (some clients set their own).
- Skip empty header values (`StringUtils.hasText`).
- Do not throw when there is no request context (scheduled/subscriber flows).

## Enable wiring (`-app` module)

```java
package pe.interbank.bcv.bacccurrentaccount.config;

import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableFeignClients(basePackages = {
        "pe.interbank.bcv.bacccurrentaccount.out.client",
        "pe.interbank.bcv.ifx"
})
@ComponentScan(basePackages = { "pe.interbank.bcv.ifx" })
public class FeignConfiguration {
}
```

Legacy `service-point-service` declares `@EnableFeignClients` directly on the application class and
uses the corporate `FeignConfigAudit` (`pe.interbank.commons.auditclient.feign.FeignConfigAudit`).

## Properties (application.yml)

```yaml
interbank:
  ads:
    httpclient:
      feign:
        customer:
          base-url: ${CUSTOMER_SERVICE_URL:http://localhost:8081}
          path: /customers
          pathQueryLegalEntity: /legal-entities
```

Base URLs and paths must be configurable; secrets/credentials come from Key Vault / env vars.

## Header propagation for external APIs

For external systems that need the full header set, pass a `Map<String, String>` as
`@RequestHeader`:

```java
@PostMapping("/service-point/create-access-point")
BieTrackingResponseDto createServicePoint(@RequestHeader Map<String, String> headers,
                                          BieServicePointCreateRequestDto requestDto);
```

## Evaluation checklist

- [ ] Client interface uses configurable `url` and `configuration = FeignConfig.class`.
- [ ] Paths are property-driven, not hardcoded.
- [ ] Payloads are records.
- [ ] `FeignConfig` exposes a `RequestInterceptor` that copies auth + trace headers.
- [ ] `@EnableFeignClients` is wired in the `-app` module.
- [ ] No secrets hardcoded.
