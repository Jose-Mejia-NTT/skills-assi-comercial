# Teams alerting configuration

## application.yml

```yaml
bcv:
  observability:
    alerting:
      enabled: true
      teams:
        webhook-url: ${bcv-webhook-url-teams}
```

## bootstrap.yml

Include the webhook secret in the Key Vault source:

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

## Security warning

Never commit the webhook URL or the observability token to git.
If you find `commons.exception.teams.webhook.endpoint` with a literal URL in `application.properties`, treat it as a security gap:
rotate the webhook and move it to Key Vault.

## Manual webhook test

```bash
curl -X POST -H "Content-Type: application/json" \
  -d '{"text":"Test alert from bcv-commons-observability"}' \
  "${BCV_WEBHOOK_URL_TEAMS}"
```
