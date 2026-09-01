---
name: bcv-spring-data-jpa-sql-server
description: >
  Generate and maintain Spring Data JPA + SQL Server persistence components for
  BCV (Business Corporate Value) Java projects. Covers entity design, repository
  interfaces, datasource configuration with HikariCP, SQL Server dialects,
  Always Encrypted, JPA auditing, schema conventions, and hexagonal/clean
  architecture placement. Follows SDD (Spec-Driven Development) spec→plan→tasks→
  implement workflow and BMAD v6 phased methodology with constitutional compliance
  gates. Produces executable code AND specification artifacts.
  Triggers: "JPA BCV", "SQL Server entity", "Spring Data repository", "HikariCP",
  "Always Encrypted", "schema BcvBacc", "SQLServer2012Dialect", "ddl-auto=none",
  "BaseEntity", "JPA auditing", "BCV persistence", "Interbank SQL Server",
  "bcv datasource", "create entity BCV", "add repository BCV"
metadata:
  version: "1.0.0"
  author: "NTT Data / BCV Architecture Team"
  language: "en"
  category: "backend"
  frameworks: ["Spec-Driven Development", "BMAD v6", "Spring Data JPA", "SQL Server"]
---

# Language handling and output policy

See [references/language-policy.md](references/language-policy.md).

---

# BCV Spring Data JPA + SQL Server Persistence Skill

Generates and maintains Spring Data JPA persistence components for BCV Java projects:

- JPA entities with SQL Server schema conventions
- Spring Data repository interfaces
- Datasource configuration (HikariCP, SQL Server dialects, Always Encrypted)
- JPA auditing (`@CreatedBy`, `@LastModifiedBy`, timestamps, `@Version`)
- Hexagonal/clean architecture placement (core vs out module)
- Unit tests for repositories and entities

This skill detects existing project scaffolding and adapts accordingly:

- **With existing hexagonal/clean scaffolding**: adds persistence components to the
  appropriate module (core for domain, out for adapters).
- **Without scaffolding**: generates a minimal Spring Boot multi-module Maven structure
  with JPA/SQL Server ready to use.

---

## Standard table columns

Every SQL table (JPA entity and its migration) must **always** include these standard columns:

`id`, `code`, `name`, `created_by`, `created_on`, `updated_by`, `updated_on`, `version`.

- `id` — primary key (generated).
- `code` — business code.
- `name` — descriptive name.
- `created_by` / `created_on` / `updated_by` / `updated_on` — audit fields (in `BaseEntity`).
- `version` — optimistic locking (`@Version`).

---

## Inputs

Accept project context in any of these forms:

- **Empty/new project** — generates minimal multi-module Maven structure
- **Existing Maven multi-module project** — detects modules (`-core`, `-out`, `-app`, `-inp`) and integrates
- **Existing single-module project** — creates JPA package structure within the module

Required information (gathered via questions):

- Base package name (e.g., `pe.interbank.bcv.baccaccountopeningreporting`)
- SQL Server schema name (e.g., `BcvBacc`, default: `dbo`)
- Entity name and fields (or a domain description to derive them)
- Whether the entity needs auditing fields (`createdBy`, `updatedBy`, timestamps, `@Version`)
- Dialect preference (`SQLServer2012Dialect` for legacy, `SQLServerDialect` for modern)
- Whether the entity needs Always Encrypted columns
- Repository methods needed (CRUD, custom queries, projections)

---

# SDD + BMAD Workflow Integration

This skill follows the **Spec-Driven Development (SDD)** methodology and the **BMAD v6**
(Breakthrough Method for Agile AI-Driven Development) phased workflow. Every execution
produces both executable code AND structured specification artifacts.

## SDD Principles Applied

| SDD Principle                       | How This Skill Applies It                                                                                                        |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| **Specifications as Lingua Franca** | Requirements gathered in Steps 0-1 become a `spec.md` artifact. Code is generated FROM the spec.                                 |
| **Executable Specifications**       | Each entity field, repository method, and config property maps to generated code. Acceptance criteria become tests.              |
| **Continuous Refinement**           | The skill validates consistency between user requirements, generated code, and architectural patterns at each step.              |
| **Research-Driven Context**         | Step 0 (Project Structure Detection) performs automated research on the existing codebase before generating.                     |
| **Bidirectional Feedback**          | Generated `application.yml`, `.env.example`, and troubleshooting notes feed back into operational reality.                       |
| **Branching for Exploration**       | Supports multiple project archetypes (new, hexagonal-multi, hexagonal-single, non-hexagonal) from the same specification intent. |

## BMAD Phased Workflow Mapping

### Fase 1: Analysis — BMAD Phase 1 (Steps 0-1)

**BMAD equivalents**: `bmad-domain-research`, `bmad-technical-research`, `bmad-product-brief`

| Step                                  | BMAD Mapping                             | Output                                                                                   |
| ------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Step 0** — Detect Project Structure | Technical + Domain Research              | Project classification, module map, package detection, existing entity patterns          |
| **Step 1** — Clarify Requirements     | Product Brief / Requirements Elicitation | Complete requirement set (entity, fields, schema, auditing, repository methods, dialect) |

**BMAD Analysis Phase Output**: `_bmad-output/analysis/spec.md`

### Fase 2: Solutioning — BMAD Phases 2-3 (Steps 2-3)

**BMAD equivalents**: `bmad-create-prd`, `bmad-create-architecture`, `bmad-create-epics-and-stories`

| Step                                         | BMAD Mapping                 | Output                                                              |
| -------------------------------------------- | ---------------------------- | ------------------------------------------------------------------- |
| **Step 2** — Generate/Update Maven Structure | ADR + PRD                    | `pom.xml` dependency management for JPA/SQL Server                  |
| **Step 3** — Generate Persistence Components | Architecture + Domain Design | Entity, repository, base entity, auditing config, datasource config |

**BMAD Solutioning Phase Output**: `_bmad-output/solutioning/plan.md`

### Fase 3: Implementation — BMAD Phase 4 (Steps 4-7)

**BMAD equivalents**: `bmad-create-story`, `bmad-dev-story`, `bmad-code-review`, `bmad-sprint-status`

| Step                                          | BMAD Mapping                 | Output                                                        |
| --------------------------------------------- | ---------------------------- | ------------------------------------------------------------- |
| **Step 4** — Generate Core Domain Layer       | Story Implementation         | Base entity, domain value objects if needed                   |
| **Step 5** — Generate OUT Persistence Adapter | Story Implementation         | JPA entity, repository, Spring Data adapter if hexagonal      |
| **Step 6** — Generate Configuration Files     | Infrastructure Configuration | `application.yml` datasource, `bootstrap.yml` Key Vault hints |
| **Step 7** — Generate Tests                   | Test-First Verification      | Entity tests, repository tests with `@DataJpaTest`            |

**BMAD Implementation Phase Output**: `_bmad-output/implementation/tasks.md`

### Quick Flow

When project structure is already detected and requirements are clear, operate in
**BMAD Quick Flow** mode — Steps 0-3 are accelerated and Steps 4-7 execute with full generation.

---

## SDD Constitutional Compliance Gates

### Gate 1: Analysis → Solutioning (After Step 1)

- [ ] All `[NEEDS CLARIFICATION]` markers resolved
- [ ] Base package confirmed (detected or user-provided)
- [ ] SQL Server schema name specified
- [ ] Entity name and primary fields confirmed
- [ ] Auditing strategy decided (`BaseEntity` or manual fields)
- [ ] Dialect selected (`SQLServer2012Dialect` vs `SQLServerDialect`)
- [ ] Module placement confirmed (for existing projects)

### Gate 2: Solutioning → Implementation (After Step 3)

**Simplicity Gate:**

- [ ] Using ≤4 modules (inp, core, out, app) — no unnecessary fragmentation
- [ ] No speculative abstractions beyond repository pattern
- [ ] Core module has zero infrastructure dependencies (only JPA annotations if needed)

**Library-First Principle:**

- [ ] Persistence components are modular and reusable across projects
- [ ] Domain layer is framework-agnostic (no Spring Data imports in pure domain if possible)

**Anti-Abstraction Gate:**

- [ ] Use Spring Data `JpaRepository` directly — no generic wrapper unless justified
- [ ] MapStruct used for boundary crossing if needed
- [ ] Single entity class per table (no dual DTO/entity without reason)

### Gate 3: Implementation → Completion (After Step 7)

**Test-First Imperative:**

- [ ] Unit tests for entity mappings
- [ ] Repository tests with `@DataJpaTest` or mocked `JpaRepository`
- [ ] Lombok + MapStruct annotation processors configured in correct order

**Integration-First Gate:**

- [ ] Datasource configuration uses realistic SQL Server + HikariCP settings
- [ ] Connection string documented in `.env.example`
- [ ] Always Encrypted columns documented with Key Vault hint

**CLI Interface Mandate:**

- [ ] All persistence operations are observable via logs (SLF4J)
- [ ] Query performance considerations noted (fetch type, lazy/eager)

---

## SDD Specification Artifacts Produced

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

### `spec.md`

Captures:

- Entity name, table, schema, fields
- User stories and acceptance criteria
- Auditing and encryption requirements
- Dialect and connection-pool requirements

### `plan.md`

Captures:

- Technology decisions (JPA, Hibernate, SQL Server driver, HikariCP)
- Module architecture justification
- Data model design decisions
- API contracts (repository methods)
- Test strategy
- File creation order

### `tasks.md`

Captures:

- Tasks derived from plan sections
- Independent tasks marked `[P]` for parallel execution
- Dependencies between tasks
- Validation checkpoints at gate boundaries

---

## Step 0 — Detect Project Structure

**Goal**: Understand the current project layout to determine generation strategy.

### 0.1 Search for Maven structure

Look for `pom.xml` files using `file_search` with pattern `**/pom.xml`.

### 0.2 Analyze Maven structure

For each `pom.xml`:

- Check if it's a parent POM with `<modules>` section
- Identify module naming patterns: `-core`, `-out`, `-app`, `-inp`
- Extract `groupId` and `artifactId` to infer base package
- Check dependencies: `spring-boot-starter-data-jpa`, `mssql-jdbc`, `HikariCP`, `lombok`, `mapstruct`
- Check parent POM reference (`ads-spring-boot-dependencies`, `bcv-commons-starter-parent`, etc.)

### 0.3 Detect existing Java packages and entities

Use `grep_search` to find package declarations:

```regex
package\s+(pe\.interbank\.[^;]+);
```

Extract the most common base package.

Search for existing `@Entity` classes to understand conventions:

```regex
@Entity|@Table\(|@Column\(|@Id|@GeneratedValue
```

### 0.4 Classify project type

| Pattern                                                 | Classification              | Strategy                                               |
| ------------------------------------------------------- | --------------------------- | ------------------------------------------------------ |
| Parent `pom.xml` + 4 modules with `-core`, `-out`, etc. | **Hexagonal multi-module**  | Add to `-core` (domain) and `-out` (entity/repository) |
| Single `pom.xml` + packages with `.core.`, `.out.`      | **Hexagonal single-module** | Add to package structure                               |
| `pom.xml` exists but no hexagonal pattern               | **Existing non-hexagonal**  | Ask user: convert or add flat JPA package              |
| No `pom.xml` or empty workspace                         | **New project**             | Generate minimal multi-module scaffolding              |

**Store in context**:

- `projectType`: `hexagonal-multi`, `hexagonal-single`, `non-hexagonal`, or `new`
- `basePackage`: detected or null
- `modules`: map of module names to paths
- `parentPom`: parent artifact reference
- `existingDialect`: detected SQL Server dialect or null
- `existingSchema`: detected schema from `@Table(schema=...)` or null

---

## Step 1 — Clarify Requirements

**Goal**: Gather all missing information from the user.

Use `vscode_askQuestions` to collect:

### 1.1 Entity name and domain

```
Question: "What is the JPA entity name?"
Example: "PaymentPromise"

Question: "Which SQL Server schema should the table use?"
Options: ["BcvBacc", "dbo", "Other (specify)"]
Default: detected schema or "dbo"

Question: "Describe the entity fields or paste the data model."
Example: "id (Long, PK), expedientNumber (String), amount (BigDecimal), status (String), createdAt (LocalDateTime)"
```

### 1.2 Auditing strategy

```
Question: "Should the entity include standard audit fields?"
Options: ["Yes - extend BaseEntity", "Yes - manual fields", "No"]
Default: "Yes - extend BaseEntity"
```

### 1.3 Dialect and version

```
Question: "Which SQL Server Hibernate dialect should be used?"
Options: ["SQLServer2012Dialect", "SQLServerDialect"]
Default: detected dialect or "SQLServer2012Dialect"
```

### 1.4 Always Encrypted

```
Question: "Does any column need SQL Server Always Encrypted?"
Options: ["Yes", "No"]
Default: "No"

Question (if Yes): "Which columns?"
Example: "accountNumber, customerName"
```

### 1.5 Repository methods

```
Question: "Which repository methods do you need?"
Options: ["CRUD only", "CRUD + custom query methods", "CRUD + custom JPQL/SQL", "Projection/lookup"]
Default: "CRUD only"
```

### 1.6 Module placement (if hexagonal-multi detected)

```
Question: "Which module should contain the JPA entity and repository?"
Options: ["{detected-out-module}" (default), "{detected-core-module}", "Other"]
Default: "{detected-out-module}"
```

---

## Step 2 — Generate or Update Maven Structure

**Goal**: Ensure proper Maven project structure with JPA/SQL Server dependencies.

### 2.1 If `projectType === 'new'`: Generate minimal structure

Create parent `pom.xml` at workspace root. Example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version>
        <relativePath/>
    </parent>

    <groupId>{basePackage}</groupId>
    <artifactId>{projectName}</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>{projectName}-core</module>
        <module>{projectName}-out</module>
        <module>{projectName}-app</module>
    </modules>

    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <mapstruct.version>1.5.5.Final</mapstruct.version>
        <lombok.version>1.18.30</lombok.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct</artifactId>
                <version>${mapstruct.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
                <scope>provided</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

Create module POMs. See `references/pom-templates.md` for full templates.

### 2.2 If `projectType === 'hexagonal-multi'`: Update existing modules

**For `-out` module** (default location for entity/repository):

- Add `spring-boot-starter-data-jpa`
- Add `mssql-jdbc`
- Add `HikariCP` if not already present
- Add `{projectName}-core` dependency
- Add `lombok` and `mapstruct` as needed
- Ensure `maven-compiler-plugin` has `annotationProcessorPaths` with Lombok BEFORE MapStruct

**For `-core` module**:

- If pure domain, keep JPA annotations minimal (only `jakarta.persistence` if BaseEntity lives there)
- Otherwise, add `jakarta.persistence-api` dependency only
- No `spring-boot-starter-data-jpa` in core

**For `-app` module**:

- Add dependency on `-out` and `-core`
- Ensure `spring-boot-starter-data-jpa` is present if not already

### 2.3 If `projectType === 'hexagonal-single' or 'non-hexagonal'`: Update single POM

Add dependencies:

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

Ensure `maven-compiler-plugin` has Lombok and MapStruct processors in correct order.

---

## Step 3 — Generate Persistence Components

### 3.1 Generate base audit entity (if requested)

**Location**:

- Multi-module: `{projectName}-core/src/main/java/{packagePath}/core/domain/entity/BaseEntity.java`
- Single-module: `src/main/java/{packagePath}/core/domain/entity/BaseEntity.java`

```java
package {basePackage}.core.domain.entity;

import jakarta.persistence.Column;
import jakarta.persistence.EntityListeners;
import jakarta.persistence.MappedSuperclass;
import jakarta.persistence.Version;
import lombok.Getter;
import lombok.Setter;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

/**
 * Base audit entity for BCV projects.
 * Provides createdBy, createdOn, updatedBy, updatedOn, and optimistic locking.
 */
@Getter
@Setter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @CreatedBy
    @Column(name = "created_by", length = 50, nullable = false, updatable = false)
    private String createdBy;

    @CreatedDate
    @Column(name = "created_on", nullable = false, updatable = false)
    private LocalDateTime createdOn;

    @LastModifiedBy
    @Column(name = "updated_by", length = 50, nullable = false)
    private String updatedBy;

    @LastModifiedDate
    @Column(name = "updated_on", nullable = false)
    private LocalDateTime updatedOn;

    @Version
    @Column(name = "version", nullable = false)
    private Long version = 0L;
}
```

### 3.2 Generate JPA entity

**Location**:

- Multi-module: `{projectName}-out/src/main/java/{packagePath}/out/database/entity/{EntityName}.java`
- Single-module: `src/main/java/{packagePath}/out/database/entity/{EntityName}.java`

```java
package {basePackage}.out.database.entity;

import {basePackage}.core.domain.entity.BaseEntity;
import jakarta.persistence.*;
import lombok.*;

import java.math.BigDecimal;
import java.time.LocalDateTime;

/**
 * JPA entity for {entityName}.
 * Table: {schema}.{tableName}
 */
@Entity
@Table(schema = "{schema}", name = "{tableName}")
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class {EntityName} extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;

    @Column(name = "code", nullable = false, length = 50)
    private String code;

    @Column(name = "name", nullable = false, length = 150)
    private String name;

    @Column(name = "expedient_number", nullable = false, length = 50)
    private String expedientNumber;

    @Column(name = "amount", precision = 18, scale = 2)
    private BigDecimal amount;

    @Column(name = "status", length = 20)
    private String status;

    @Column(name = "processed_at")
    private LocalDateTime processedAt;

    // Add additional fields as required by the spec
}
```

### 3.3 Generate Spring Data repository

**Location**:

- Multi-module: `{projectName}-out/src/main/java/{packagePath}/out/database/repository/{EntityName}Repository.java`
- Single-module: `src/main/java/{packagePath}/out/database/repository/{EntityName}Repository.java`

```java
package {basePackage}.out.database.repository;

import {basePackage}.out.database.entity.{EntityName};
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

/**
 * Spring Data repository for {EntityName}.
 */
@Repository
public interface {EntityName}Repository extends JpaRepository<{EntityName}, Long> {

    Optional<{EntityName}> findByExpedientNumber(String expedientNumber);

    List<{EntityName}> findByStatusOrderByProcessedAtDesc(String status);

    @Query("SELECT e FROM {EntityName} e WHERE e.status = :status AND e.processedAt >= :since")
    List<{EntityName}> findActiveSince(@Param("status") String status, @Param("since") LocalDateTime since);
}
```

### 3.4 Generate JPA auditing configuration

**Location**:

- Multi-module: `{projectName}-app/src/main/java/{packagePath}/app/config/JpaAuditingConfig.java`
- Single-module: `src/main/java/{packagePath}/app/config/JpaAuditingConfig.java`

```java
package {basePackage}.app.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.domain.AuditorAware;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

import java.util.Optional;

/**
 * JPA auditing configuration.
 * Provides current user from JWT context.
 */
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorAware")
public class JpaAuditingConfig {

    @Bean
    public AuditorAware<String> auditorAware() {
        // Replace with JWT-aware implementation (e.g., SecurityContextHolder)
        return () -> Optional.ofNullable("system");
    }
}
```

---

## Step 4 — Generate Configuration Files

### 4.1 Datasource configuration

**Location**:

- Multi-module: `{projectName}-app/src/main/resources/application.yml`
- Single-module: `src/main/resources/application.yml`

```yaml
spring:
  application:
    name: {projectName}
  datasource:
    url: ${SQLSERVER_URL:jdbc:sqlserver://localhost:1433;databaseName=BCV;encrypt=true;trustServerCertificate=false}
    username: ${SQLSERVER_USERNAME}
    password: ${SQLSERVER_PASSWORD}
    driver-class-name: com.microsoft.sqlserver.jdbc.SQLServerDriver
    type: com.zaxxer.hikari.HikariDataSource
    hikari:
      pool-name: {projectName}-pool
      maximum-pool-size: 30
      minimum-idle: 5
      connection-timeout: 25000
      idle-timeout: 10000
      leak-detection-threshold: 20000
      auto-commit: false
  jpa:
    database-platform: org.hibernate.dialect.{dialect}
    hibernate:
      ddl-auto: none
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.{dialect}
        implicit_naming_strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
        physical_naming_strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
        use_nationalized_character_data: true
        temp:
          use_jdbc_metadata_defaults: false
```

### 4.2 `.env.example`

```bash
# SQL Server configuration
SQLSERVER_URL=jdbc:sqlserver://your-server.database.windows.net:1433;database=your-db;encrypt=true;trustServerCertificate=false
SQLSERVER_USERNAME=your-username
SQLSERVER_PASSWORD=your-password

# Azure Key Vault (optional, for Always Encrypted column master keys)
KEYVAULT_URI=https://your-keyvault.vault.azure.net/
AZURE_CLIENT_ID=your-client-id
AZURE_CLIENT_SECRET=your-client-secret
AZURE_TENANT_ID=your-tenant-id
```

---

## Step 5 — Generate Tests

### 5.1 Entity mapping test

```java
package {basePackage}.out.database.entity;

import org.junit.jupiter.api.Test;

import java.math.BigDecimal;
import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.assertThat;

class {EntityName}Test {

    @Test
    void shouldBuildEntity() {
        var entity = {EntityName}.builder()
            .expedientNumber("EXP-12345")
            .amount(BigDecimal.valueOf(1000.50))
            .status("PENDING")
            .processedAt(LocalDateTime.now())
            .build();

        assertThat(entity.getExpedientNumber()).isEqualTo("EXP-12345");
        assertThat(entity.getAmount()).isEqualByComparingTo(BigDecimal.valueOf(1000.50));
        assertThat(entity.getStatus()).isEqualTo("PENDING");
    }
}
```

### 5.2 Repository test

```java
package {basePackage}.out.database.repository;

import {basePackage}.out.database.entity.{EntityName};
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

import java.math.BigDecimal;
import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
class {EntityName}RepositoryTest {

    @Autowired
    private {EntityName}Repository repository;

    @Test
    void shouldSaveAndFindByExpedientNumber() {
        var entity = {EntityName}.builder()
            .expedientNumber("EXP-001")
            .amount(BigDecimal.TEN)
            .status("PENDING")
            .processedAt(LocalDateTime.now())
            .build();

        repository.save(entity);

        var found = repository.findByExpedientNumber("EXP-001");

        assertThat(found).isPresent();
        assertThat(found.get().getStatus()).isEqualTo("PENDING");
    }
}
```

---

## Output Format

After generation, show the complete file tree:

```
✅ Generated Spring Data JPA + SQL Server Persistence Components

📁 Project structure:

{projectName}/
├── pom.xml
├── .env.example
├── {projectName}-core/
│   └── src/main/java/{packagePath}/core/domain/entity/
│       └── BaseEntity.java
├── {projectName}-out/
│   └── src/main/java/{packagePath}/out/database/
│       ├── entity/
│       │   └── {EntityName}.java
│       └── repository/
│           └── {EntityName}Repository.java
└── {projectName}-app/
    └── src/main/
        ├── java/{packagePath}/app/config/
        │   └── JpaAuditingConfig.java
        └── resources/
            └── application.yml

📁 Test files:
├── {projectName}-out/src/test/java/{packagePath}/out/database/entity/{EntityName}Test.java
└── {projectName}-out/src/test/java/{packagePath}/out/database/repository/{EntityName}RepositoryTest.java

📋 Specification artifacts:
└── _bmad-output/
    ├── analysis/spec.md
    ├── solutioning/plan.md
    └── implementation/tasks.md
```

---

## Troubleshooting Notes

### Dialect mismatch

**Error**: `SQLServerException` with unsupported feature

**Solution**: Verify dialect matches SQL Server version. Use `SQLServer2012Dialect` for legacy compatibility.

### Always Encrypted column master key not found

**Error**: `Microsoft.Data.SqlClient.SqlException` about key path

**Solution**: Ensure `columnEncryptionSetting=Enabled` is in JDBC URL and Key Vault provider is registered.

### Hikari pool exhausted

**Error**: `HikariPool-1 - Thread starvation or clock leap`

**Solution**: Increase `maximum-pool-size`, review connection leaks, enable `leak-detection-threshold`.

### MapStruct mapper not generated

**Error**: `Cannot autowire {EntityName}Mapper`

**Solution**: Verify Lombok comes BEFORE MapStruct in `annotationProcessorPaths`.

### `ddl-auto=none` requires table pre-existence

**Error**: `SQLServerException: Invalid object name`

**Solution**: Tables must be created externally. This skill does not generate DDL.

---

## Reference Documents

Load from `references/` based on context:

| Reference                        | Content                                            | When to Load                             |
| -------------------------------- | -------------------------------------------------- | ---------------------------------------- |
| `references/pom-templates.md`    | Maven POM templates for core, out, app modules     | When generating new multi-module project |
| `references/configuration.md`    | Datasource + HikariCP + JPA properties guide       | When generating `application.yml`        |
| `references/always-encrypted.md` | Always Encrypted setup and JDBC URL hints          | When Always Encrypted requested          |
| `references/auditing.md`         | `@EnableJpaAuditing`, `AuditorAware`, `BaseEntity` | When auditing requested                  |

---

## Offer Next Action

After showing the output, ask:

> I've generated the Spring Data JPA + SQL Server persistence components.
>
> Would you like me to:
>
> 1. **Customize the entity fields** with more domain-specific attributes?
> 2. **Add a custom query or projection** to the repository?
> 3. **Generate a MapStruct mapper** for entity ↔ DTO conversion?
> 4. **Add a service/use case** layer to consume the repository?
> 5. **Generate integration tests** with Testcontainers SQL Server?

Wait for user confirmation before proceeding.
