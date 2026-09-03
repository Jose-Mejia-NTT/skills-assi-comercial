# Feign error decoder (BACC)

Use this reference to map external-service failures to domain exceptions and to avoid leaking
sensitive data in logs.

## Why a custom ErrorDecoder

By default OpenFeign throws a generic `FeignException`. BACC services map 4xx/5xx to a typed
`ExternalServiceException` so upstream callers can react (retry, fallback, business rule).

## Pattern A — status-based mapping (PLM)

Maps each status code to a specific message; sanitizes the body before logging.

```java
package pe.interbank.bcv.baccpartylifecyclemanagement.config.error.external.feign;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import feign.Response;
import feign.codec.ErrorDecoder;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import pe.interbank.bcv.baccpartylifecyclemanagement.config.error.external.ExternalServiceException;

import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.util.List;

public class CustomFeignErrorDecoder implements ErrorDecoder {

    private static final Logger log = LoggerFactory.getLogger(CustomFeignErrorDecoder.class);
    private final String serviceName;
    private final ObjectMapper objectMapper;
    private final ErrorDecoder defaultDecoder = new Default();

    public CustomFeignErrorDecoder(String serviceName, ObjectMapper objectMapper) {
        this.serviceName = serviceName;
        this.objectMapper = objectMapper;
    }

    @Override
    public Exception decode(String methodKey, Response response) {
        int status = response.status();
        String url = response.request() != null ? response.request().url() : "unknown";
        String method = response.request() != null ? response.request().httpMethod().name() : "?";
        String body = readBody(response);

        log.error("[FEIGN] Upstream error [service={}] [{}] [{}] [status={}] | body={}",
                serviceName, method, url, status, sanitize(body));

        return switch (status) {
            case 400 -> ExternalServiceException.fromHttp(serviceName, method, url, 400, body, null);
            case 401 -> ExternalServiceException.fromHttp(serviceName, method, url, 401, "Authentication failed.", null);
            case 403 -> ExternalServiceException.fromHttp(serviceName, method, url, 403, "Insufficient permissions.", null);
            case 404 -> ExternalServiceException.fromHttp(serviceName, method, url, 404, "Resource not found.", null);
            case 409 -> ExternalServiceException.fromHttp(serviceName, method, url, 409, body, null);
            case 429 -> ExternalServiceException.fromHttp(serviceName, method, url, 429, "Rate limit exceeded.", null);
            case 500 -> ExternalServiceException.fromHttp(serviceName, method, url, 500, "Internal error.", null);
            case 503 -> ExternalServiceException.fromHttp(serviceName, method, url, 503, "Unavailable.", null);
            case 504 -> ExternalServiceException.timeout(serviceName, url);
            default  -> defaultDecoder.decode(methodKey, response);
        };
    }

    private String readBody(Response response) {
        if (response.body() == null) return "<empty body>";
        try (InputStream is = response.body().asInputStream()) {
            return new String(is.readAllBytes(), StandardCharsets.UTF_8);
        } catch (IOException e) {
            return "<unreadable body>";
        }
    }

    private String sanitize(String body) {
        if (body == null || body.isBlank()) return "<empty>";
        try {
            JsonNode node = objectMapper.readTree(body);
            ((com.fasterxml.jackson.databind.node.ObjectNode) node)
                    .remove(List.of("token", "access_token", "password", "secret", "pan", "cvv", "pin"));
            return node.toString();
        } catch (Exception e) {
            return body.length() > 500 ? body.substring(0, 500) + "…[truncated]" : body;
        }
    }
}
```

## Pattern B — body-based mapping (customer)

Parses the error body into a structured DTO and maps to `ExternalServiceException`.

```java
package pe.interbank.bcv.bacccustomer.out.client.config;

import com.fasterxml.jackson.databind.ObjectMapper;
import feign.Response;
import feign.codec.ErrorDecoder;
import lombok.extern.slf4j.Slf4j;
import pe.interbank.bcv.bacccustomer.out.exception.ExternalServiceException;

import java.io.InputStream;
import java.nio.charset.StandardCharsets;

@Slf4j
public class FeignCustomErrorDecoder implements ErrorDecoder {

    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public Exception decode(String methodKey, Response response) {
        try (InputStream body = response.body().asInputStream()) {
            String json = new String(body.readAllBytes(), StandardCharsets.UTF_8);
            ExternalErrorResponse error = objectMapper.readValue(json, ExternalErrorResponse.class);
            return new ExternalServiceException(error.getDetail(), error.getStatus(), error.getErrorCode());
        } catch (Exception e) {
            log.error("Error decoding Feign error body", e);
            return new ExternalServiceException("Unexpected error calling external service",
                    response.status(), "EXTERNAL_SERVICE_ERROR");
        }
    }
}
```

## Registration

Expose the decoder as a `@Bean` in `FeignConfig`:

```java
@Configuration
public class FeignConfig {

    @Bean
    public ErrorDecoder errorDecoder(ObjectMapper objectMapper) {
        return new CustomFeignErrorDecoder("my-service", objectMapper);
    }
}
```

## Rules

1. Always map non-2xx to a domain exception; never let raw `FeignException` reach the core.
2. Sanitize sensitive fields (`token`, `password`, `pan`, `cvv`, `pin`) before logging.
3. Treat 503/504 (and timeouts) as retryable; 4xx as not retryable.
4. Read the body once; use try-with-resources.
5. Keep the decoder stateless (constructor args only).
