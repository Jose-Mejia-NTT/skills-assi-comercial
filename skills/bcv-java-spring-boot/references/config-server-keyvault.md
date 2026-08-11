# Config Server and Key Vault Secret Flow

Use this reference for secure externalized configuration.

## Security baseline

- No plain credentials in source control.
- Secret values must be sourced from environment variables, Config Server, or Key Vault.
- Secret names should be explicit and stable across environments.

## Suggested bootstrap pattern

```yaml
spring:
  cloud:
    azure:
      keyvault:
        secret:
          property-sources-enabled: ${KEYVAULT_ENABLED:true}
          property-sources[0]:
            endpoint: ${KEYVAULT_URI}
            credential:
              client-id: ${KEYVAULT_CLIENT_ID}
              client-secret: ${KEYVAULT_CLIENT_KEY}
            profile:
              tenant-id: ${TENANT_ID}
            secret-keys: "bcv-observability-token,bcv-webhook-url-teams"
```

Adjust secret names to the target BCV project.

## Validation checklist

1. Required environment variables are present at runtime.
2. Key Vault access permissions are granted to app identity.
3. Missing secret fails with actionable error message.
4. No secret literal exists in repository history.

## If external config is unavailable

Apply mock-first and fallback strategy:

- Use local stubs for external services.
- Keep secret placeholders as environment-based values.
- Document migration criteria from mock to real endpoints.
