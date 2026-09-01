# BACC Spring Boot conventions

Used by the 7 `bcv-bacc-*` services. Use this reference for parent/BOM alignment, launchers,
secret wiring and datasource configuration in the BACC ecosystem.

## Parent / BOM

| Service | Parent | Version | Java |
| --- | --- | --- | --- |
| 6 of 7 BACC services | `pe.interbank.ads:ads-spring-boot-dependencies` | 3.5.3 / 3.5.10 | 21 |
| `service-point-service` | `pe.interbank.bcv.commons:bcv-commons-pomparent` | 2.0.0 | 17 |

Do not change the parent of `service-point-service` casually; it is the legacy exception.

## Module layout

```
bcv-bacc-<service>-service/
├── bcv-bacc-<service>-app/      # bootstrap, config, launcher, application.yml/bootstrap.yml
├── bcv-bacc-<service>-core/     # domain, use cases, ports
├── bcv-bacc-<service>-input/    # REST controllers, subscribers (entry points)
└── bcv-bacc-<service>-output/   # adapters, repositories, publishers, Feign clients
```

`service-point-service` is a single flat module (no hexagonal split).

## Launcher (app module)

Launcher class name is `Bacc*AppLauncher` or `Bacc*Application` in package `pe.interbank.bcv.bacc<service>`:

```java
package pe.interbank.bcv.baccaccountopeningreporting;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication(scanBasePackages = "pe.interbank.bcv.baccaccountopeningreporting")
public class BaccAccountreportingOpeningAppLauncher {
    public static void main(String[] args) {
        SpringApplication.run(BaccAccountreportingOpeningAppLauncher.class, args);
    }
}
```

Known launchers:

- `BaccAccountreportingOpeningAppLauncher`
- `BaccChannelActivityAppLauncher`
- `BaccComplianceApplication`
- `BaccCurrentAccountAppLauncher`
- `BaccCustomerAppLauncher`
- `BcvBaccServicePointServiceApplication` (legacy)

## bootstrap.yml (Config Server + Key Vault)

BACC uses `interbank.ads.security` (Azure Key Vault via `vaults`) + `interbank.ads.secrets`,
mapped to `${ibk-nr_*}` placeholders. Do NOT use the generic `spring.cloud.azure.keyvault` block.

```yaml
server:
  port: ${PORT:8080}
spring:
  application:
    name: bcv-bacc-account-opening-reporting-service
  profiles:
    active: ${PROFILE:dev}
  cloud:
    consul:
      config:
        enabled: false
    config:
      uri: ${CONFIG_SERVER_URI}
      fail-fast: ${CONFIG_SERVER_FAIL_FAST:true}
      enabled: ${CONFIG_SERVER_ENABLED:true}
      label: ${LABEL:develop}

interbank:
  ads:
    security:
      property-source-name: bcv-bacc-account-opening-reporting-service-secrets
      vaults:
        ibk-nr:
          type: azure
          url: ${KEYVAULT_URI}
          clientId: ${KEYVAULT_CLIENT_ID}
          clientSecret: ${KEYVAULT_CLIENT_KEY}
          tenantId: ${TENANT_ID}
    secrets:
      database:
        connectionString: ${ibk-nr_bcv-bacc-datasource-url-jdbc}
      serviceBus:
        connectionString: ${ibk-nr_bcv-bacc-sb-connection-string}
      cosmos:
        url: ${ibk-nr_bcv-bacc-cosmos-hub-url}
        key: ${ibk-nr_bcv-bacc-cosmos-hub-key}
      storage:
        bcw:
          name: ${ibk-nr_bcw-bacc-blob-storage-account-name}
          key: ${ibk-nr_bcw-bacc-blob-storage-account-key}
```

## Datasource (application.yml)

BACC uses `interbank.ads.persistence-sql.sql-data-sources.<ds>` (not `spring.datasource`):

```yaml
interbank:
  ads:
    persistence-sql:
      sql-data-sources:
        clientDs:
          connectionString: ${interbank.ads.secrets.database.connectionString}
          driverClassName: com.microsoft.sqlserver.jdbc.SQLServerDriver
          dialect: org.hibernate.dialect.SQLServerDialect
          entitiesPackage: "pe.interbank.bcv.baccaccountopeningreporting.out.database.entity"
          showSql: true
          minIdle: 1
          maxPoolSize: 10
          connectionTimeout: 30000
          idleTimeout: 600000
          maxLifetime: 900000
```

## Observability dependency

`bcv-commons-observability` is added to the `-input` (and sometimes `-output`) module:

```xml
<dependency>
    <groupId>pe.interbank.bcv.commons</groupId>
    <artifactId>bcv-commons-observability</artifactId>
</dependency>
```

Annotations `@ObservableController`, `@ObservableService`, `@ObservableOperation` come from
`pe.interbank.bcv.observability.annotation.*`. See the `bcv-commons-observability` skill.

## App module dependencies (typical)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>pe.interbank.ads.commons</groupId>
    <artifactId>ads-spring-boot-starter-security-configuration</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
<dependency>
    <groupId>com.azure.spring</groupId>
    <artifactId>spring-cloud-azure-starter-keyvault-secrets</artifactId>
</dependency>
```

## Validation checklist

- [ ] Correct BACC parent identified (`ads-spring-boot-dependencies` vs `bcv-commons-pomparent`).
- [ ] Java version matches the service (21 vs 17 for service-point).
- [ ] `scanBasePackages` matches the service base package (`pe.interbank.bcv.bacc<service>`).
- [ ] Secrets wired via `interbank.ads.security` + `interbank.ads.secrets`, never hardcoded.
- [ ] Datasource uses `interbank.ads.persistence-sql.sql-data-sources`, not `spring.datasource`.
- [ ] `bootstrap.yml` handles bootstrap/config/keyvault; `application.yml` handles runtime.
