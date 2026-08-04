# JPA Auditing Reference

Base entity and auditing configuration for BCV Spring Data JPA + SQL Server projects.

## BaseEntity

```java
package pe.interbank.bcv.disb.business.core.domain.entity;

import jakarta.persistence.Column;
import jakarta.persistence.EntityListeners;
import jakarta.persistence.MappedSuperclass;
import jakarta.persistence.Version;
import java.time.LocalDateTime;
import lombok.Getter;
import lombok.Setter;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter
@Setter
public abstract class BaseEntity {

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @CreatedBy
    @Column(name = "created_by", length = 50, nullable = false, updatable = false)
    private String createdBy;

    @LastModifiedDate
    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt;

    @LastModifiedBy
    @Column(name = "updated_by", length = 50, nullable = false)
    private String updatedBy;

    @Version
    @Column(name = "version", nullable = false)
    private Long version = 0L;
}
```

## Auditing configuration

```java
package pe.interbank.bcv.disb.business.app.config;

import java.util.Optional;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.domain.AuditorAware;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorAware")
public class JpaAuditingConfig {

    @Bean
    public AuditorAware<String> auditorAware() {
        return () -> Optional.ofNullable(
            // Replace with real user context (SecurityContext, request header, etc.)
            "system"
        );
    }
}
```

## Entity extending BaseEntity

```java
@Entity
@Table(name = "payment_promise", schema = "Disbursement")
@Data
@EqualsAndHashCode(callSuper = true)
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class PaymentPromise extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false, updatable = false)
    private Long id;

    @Column(name = "expedient_number", length = 50, nullable = false, unique = true)
    private String expedientNumber;

    @Column(name = "amount", precision = 18, scale = 2, nullable = false)
    private BigDecimal amount;

    @Column(name = "status", length = 20, nullable = false)
    private String status;

    @Column(name = "processed_at")
    private LocalDateTime processedAt;
}
```

## Notes

- `BaseEntity` lives in the `-core` module because it is part of the domain model.
- `JpaAuditingConfig` lives in the `-app` module because it wires Spring infrastructure.
- `AuditorAware` must return a non-empty `Optional`; otherwise `@CreatedBy` will be `null`.
- For `LocalDateTime` auditing with SQL Server, ensure `datetime2` or `datetimeoffset` columns are used.
- `version` is used for optimistic locking and must be mapped to a `BIGINT` column.
