# SQL Server Datasource Configuration

Reference for `application.yml` / `application.properties` configuration used by BCV Spring Data JPA + SQL Server projects.

## Standard `application.yml`

```yaml
spring:
  application:
    name: ${APPLICATION_NAME:my-bcv-service}

  datasource:
    url: ${SQLSERVER_URL}
    username: ${SQLSERVER_USERNAME}
    password: ${SQLSERVER_PASSWORD}
    driver-class-name: com.microsoft.sqlserver.jdbc.SQLServerDriver
    type: com.zaxxer.hikari.HikariDataSource
    hikari:
      pool-name: ${APPLICATION_NAME:my-bcv-service}-pool
      maximum-pool-size: 30
      minimum-idle: 5
      connection-timeout: 25000
      idle-timeout: 10000
      max-lifetime: 1800000
      leak-detection-threshold: 20000
      auto-commit: false
      connection-test-query: SELECT 1
      validation-timeout: 5000

  jpa:
    database-platform: org.hibernate.dialect.SQLServer2012Dialect
    hibernate:
      ddl-auto: none
    show-sql: false
    open-in-view: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.SQLServer2012Dialect
        implicit_naming_strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
        physical_naming_strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
        use_nationalized_character_data: true
        format_sql: true
        jdbc:
          batch_size: 50
        order_inserts: true
        order_updates: true
        connection:
          isolation: 2
```

## `.env.example`

```bash
# SQL Server
SQLSERVER_URL=jdbc:sqlserver://myserver.database.windows.net:1433;database=mydb;encrypt=true;trustServerCertificate=false;loginTimeout=30
SQLSERVER_USERNAME=myuser
SQLSERVER_PASSWORD=mypassword

# Application
APPLICATION_NAME=my-bcv-service
```

## Notes

- `ddl-auto: none` is mandatory for BCV production environments.
- `PhysicalNamingStrategyStandardImpl` keeps physical column names exactly as written in `@Column(name = "...")`.
- `ImplicitNamingStrategyLegacyJpaImpl` uses legacy JPA implicit naming.
- For Azure SQL Database, add `;encrypt=true;trustServerCertificate=false`.
- For Always Encrypted, add `;columnEncryptionSetting=Enabled` to the URL.
- HikariCP `auto-commit: false` matches the transaction boundary managed by Spring `@Transactional`.
