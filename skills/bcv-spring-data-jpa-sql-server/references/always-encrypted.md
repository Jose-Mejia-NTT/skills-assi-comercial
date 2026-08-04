# SQL Server Always Encrypted

This reference describes how to configure and use SQL Server Always Encrypted with Spring Data JPA + Hibernate.

## When to use

Use Always Encrypted when sensitive columns (e.g., account number, document number, personal data) must be encrypted at rest and in transit, and only decrypted by authorized clients.

## Configuration

### 1. Add Always Encrypted to JDBC URL

```yaml
spring:
  datasource:
    url: jdbc:sqlserver://myserver.database.windows.net:1433;database=mydb;columnEncryptionSetting=Enabled;encrypt=true;trustServerCertificate=false
```

### 2. Column annotation

```java
import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.Table;
import lombok.Data;
import lombok.EqualsAndHashCode;

@Entity
@Table(name = "bank_account", schema = "Disbursement")
@Data
@EqualsAndHashCode(callSuper = true)
public class BankAccount extends BaseEntity {

    @Column(name = "account_number", length = 50, nullable = false)
    private String accountNumber;

    @Column(name = "account_holder_name", length = 200)
    private String accountHolderName;
}
```

### 3. Maven dependency

Always Encrypted requires the `mssql-jdbc` artifact with AE support (included by default).

```xml
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>mssql-jdbc</artifactId>
    <version>12.6.1.jre11</version>
</dependency>
```

## Key store providers

### Azure Key Vault

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-security-keyvault-keys</artifactId>
    <version>4.8.3</version>
</dependency>
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>adal4j</artifactId>
    <version>1.6.7</version>
</dependency>
```

```java
import com.microsoft.sqlserver.jdbc.SQLServerColumnEncryptionAzureKeyVaultProvider;
import com.microsoft.sqlserver.jdbc.SQLServerConnection;

SQLServerColumnEncryptionAzureKeyVaultProvider akvProvider =
    new SQLServerColumnEncryptionAzureKeyVaultProvider(clientId, clientKey);
SQLServerConnection.registerColumnEncryptionKeyStoreProviders(
    Collections.singletonMap("AZURE_KEY_VAULT", akvProvider)
);
```

### Windows Certificate Store

```java
import com.microsoft.sqlserver.jdbc.SQLServerColumnEncryptionCertificateStoreProvider;

SQLServerColumnEncryptionCertificateStoreProvider certStoreProvider =
    new SQLServerColumnEncryptionCertificateStoreProvider();
SQLServerConnection.registerColumnEncryptionKeyStoreProviders(
    Collections.singletonMap("MSSQL_CERTIFICATE_STORE", certStoreProvider)
);
```

## Column-level setup (DBA)

Always Encrypted requires:

1. Column Master Key (CMK) in a key store.
2. Column Encryption Key (CEK) protected by CMK.
3. Columns defined with `ENCRYPTED WITH (COLUMN_ENCRYPTION_KEY = MyCEK, ENCRYPTION_TYPE = DETERMINISTIC, ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256')`.

```sql
CREATE TABLE Disbursement.bank_account (
    id BIGINT PRIMARY KEY,
    account_number VARCHAR(50) ENCRYPTED WITH (
        COLUMN_ENCRYPTION_KEY = DisbursementCEK,
        ENCRYPTION_TYPE = DETERMINISTIC,
        ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256'
    ) NOT NULL,
    account_holder_name NVARCHAR(200) ENCRYPTED WITH (
        COLUMN_ENCRYPTION_KEY = DisbursementCEK,
        ENCRYPTION_TYPE = RANDOMIZED,
        ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256'
    )
);
```

## Querying encrypted columns

- **Deterministic encryption**: supports equality lookups (`=`, `IN`), grouping, joins.
- **Randomized encryption**: does not support queries; use only for retrieval by primary key.

```java
// Works for DETERMINISTIC only
List<BankAccount> findByAccountNumber(String accountNumber);
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `Operand type clash: nvarchar is incompatible with ...` | Driver not sending parameter as encrypted | Add `columnEncryptionSetting=Enabled` to URL |
| `Encryption scheme mismatch` | Column metadata changed or key missing | Verify CEK/CMK and column definition |
| `Key store provider not found` | Provider not registered | Register Azure Key Vault or certificate provider at startup |
| Decryption fails | Application identity lacks key permissions | Grant `get`, `unwrapKey`, `verify`, `sign` permissions in Key Vault |

## Important notes

- Do not use `PhysicalNamingStrategy` that transforms column names for encrypted columns; column names must match the DB metadata exactly.
- Always Encrypted with secure enclaves requires different URL parameters and server-side attestation.
- For local development, use deterministic encryption and a self-signed certificate in the local certificate store.
