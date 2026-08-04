# BCV Spring Data JPA + SQL Server Persistence Skill

Genera y mantiene componentes de persistencia con **Spring Data JPA + SQL Server** para proyectos Java del ecosistema BCV (Business Corporate Value), siguiendo las convenciones de Interbank y patrones de arquitectura hexagonal/clean.

## ¿Qué cubre?

Este skill se enfoca específicamente en:

- **Entidades JPA** con convenciones BCV (`@Table(schema = "Disbursement")`, `PhysicalNamingStrategyStandardImpl`, etc.)
- **Repositorios Spring Data** (`JpaRepository`, queries derivados, JPQL)
- **Configuración de datasource** con HikariCP y SQL Server
- **Auditoría JPA** (`@CreatedBy`, `@LastModifiedBy`, timestamps, `@Version`)
- **SQL Server Always Encrypted** (cuando aplica)
- **Colocación en arquitectura hexagonal/clean** (`core` para dominio, `out` para adaptadores JPA)
- **Tests** con `@DataJpaTest` y mocks

## ¿Qué NO cubre?

- Migraciones de esquema (Flyway/Liquibase) — BCV usa `ddl-auto=none` y esquemas gestionados externamente.
- Lógica de negocio compleja — solo genera la capa de persistencia.
- Integración con otros motores de base de datos — solo SQL Server.

## Arquitecturas soportadas

### 1. Proyecto nuevo

Genera una estructura mínima multi-módulo:

```
my-bcv-service/
├── pom.xml
├── .env.example
├── my-bcv-service-core/
│   └── src/main/java/.../core/domain/entity/BaseEntity.java
├── my-bcv-service-out/
│   └── src/main/java/.../out/database/
│       ├── entity/{EntityName}.java
│       └── repository/{EntityName}Repository.java
└── my-bcv-service-app/
    └── src/main/
        ├── java/.../app/config/JpaAuditingConfig.java
        └── resources/application.yml
```

### 2. Proyecto hexagonal existente

Detecta módulos `-core`, `-out`, `-app` y agrega:
- `BaseEntity` en `-core`
- Entidad y repository en `-out`
- Configuración de auditoría y datasource en `-app`

### 3. Proyecto monolítico existente

Genera paquetes `core/domain/entity/` y `out/database/` dentro del módulo único.

## Frases de activación

- "Crea una entidad JPA para BCV"
- "Genera un repository SQL Server"
- "Configura HikariCP para SQL Server"
- "Agrega auditoría JPA a mi entidad"
- "Entidad con Always Encrypted"
- "JPA BCV schema Disbursement"
- "Spring Data repository para {Entity}"

## Ejemplo de uso

```
Usuario: "Necesito crear una entidad JPA PaymentPromise en el schema Disbursement,
          con campos expedientNumber, amount, status y processedAt. Usa arquitectura
          hexagonal y el package pe.interbank.bcv.disb.business."

Agente: [Detecta estructura del proyecto]
        [Hace preguntas para completar información]
        [Genera BaseEntity, PaymentPromise, PaymentPromiseRepository, config, tests]
        [Muestra árbol de archivos generados]
        [Ofrece siguientes pasos]
```

## Preguntas que hará el skill

1. **¿Nombre de la entidad?** (ej. `PaymentPromise`)
2. **¿Schema de SQL Server?** (ej. `Disbursement`, `H2H`, `dbo`)
3. **¿Campos de la entidad?** descripción o lista
4. **¿Auditoría?** extender `BaseEntity` o campos manuales
5. **¿Dialecto Hibernate?** `SQLServer2012Dialect` o `SQLServerDialect`
6. **¿Always Encrypted?** y en qué columnas
7. **¿Métodos del repository?** CRUD, queries derivados, JPQL
8. **¿Ubicación del módulo?** (solo si hay estructura hexagonal)

## Dependencias que agrega

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>mssql-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
</dependency>
```

## Configuración generada

### `application.yml`

```yaml
spring:
  datasource:
    url: ${SQLSERVER_URL}
    username: ${SQLSERVER_USERNAME}
    password: ${SQLSERVER_PASSWORD}
    driver-class-name: com.microsoft.sqlserver.jdbc.SQLServerDriver
    type: com.zaxxer.hikari.HikariDataSource
    hikari:
      pool-name: my-bcv-service-pool
      maximum-pool-size: 30
      minimum-idle: 5
      connection-timeout: 25000
      idle-timeout: 10000
      leak-detection-threshold: 20000
      auto-commit: false
  jpa:
    database-platform: org.hibernate.dialect.SQLServer2012Dialect
    hibernate:
      ddl-auto: none
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.SQLServer2012Dialect
        implicit_naming_strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
        physical_naming_strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
        use_nationalized_character_data: true
```

## Artefactos SDD/BMAD

El skill genera en `_bmad-output/`:

```
_bmad-output/
├── analysis/
│   └── spec.md
├── solutioning/
│   ├── plan.md
│   └── architecture.md
└── implementation/
    └── tasks.md
```

## Estructura del repositorio de skills

```
skills/bcv-spring-data-jpa-sql-server/
├── SKILL.md                          # Instrucciones para el agente
├── README.md                         # Este archivo
├── evals/
│   ├── evals.json                    # Casos de prueba
│   └── fixtures/
│       └── pom-example.xml
└── references/
    ├── pom-templates.md
    ├── configuration.md
    ├── always-encrypted.md
    └── auditing.md
```

## Evaluación

Este skill incluye casos de prueba que verifican:

1. Generación desde cero de entidad + repository
2. Integración con proyecto hexagonal existente
3. Configuración de datasource y HikariCP
4. Generación de `BaseEntity` con auditoría

## Solución de problemas

Ver **Troubleshooting Notes** en [`SKILL.md`](SKILL.md) para:
- Dialect mismatch
- Always Encrypted key path
- Hikari pool exhausted
- MapStruct mapper no generado
- `ddl-auto=none` requiere tabla pre-existente

## Notas importantes

⚠️ **Seguridad:** Nunca commitees el archivo `.env` con credenciales reales. Usa Azure Key Vault para producción.

⚠️ **Orden de annotation processors:** Lombok **DEBE** estar antes que MapStruct.

⚠️ **Esquema:** Las tablas deben existir previamente (`ddl-auto=none`). Este skill no genera DDL.
