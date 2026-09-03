# commons.audit pattern

Used in `bcv-h2h-document-management-service` and `bcv-h2h-expedient-management-service` for audit events.

The project publishes audit events automatically via `commons.audit`:

```yaml
commons:
  audit:
    enabled: true
    topic:
      connection: ${interbank.ads.bcv-secrets.serviceBus.connectionString}
      label: audit.audit
    publisher.topicName: bcv.audit
    excluded:
      uris:
        - uriPattern: .*
          methods:
            - GET
```

Changes here normally involve enabling/disabling audit, adding exclusions or rotating the connection string in Key Vault.
