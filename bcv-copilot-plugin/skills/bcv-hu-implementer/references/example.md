# Example — Implementing "Agregar oficina registral en canal BCW"

## Input

```text
/hu-implement hu-technical-refinement/HU-AGREGAR-OFICINA-REGISTRAL-BCW-refined-202608261830.md --apply
```

## Step 1: Read DHU and context

The skill reads:

- `hu-technical-refinement/HU-AGREGAR-OFICINA-REGISTRAL-BCW-refined-202608261830.md`
- `.context/hu-agregar-oficina-registral-canal-bcw.md`

From the DHU it extracts:

- Primary service: `bcv-bacc-party-lifecycle-management-service`
- Secondary service: `bcv-bacc-channel-activity-service`
- Field: `registryOffice`
- New table: `registry_office`

## Step 2: Create feature branches

```bash
cd bcv-bacc-party-lifecycle-management-service
git checkout -b feature/HU-AGREGAR-OFICINA-REGISTRAL-BCW

cd bcv-bacc-channel-activity-service
git checkout -b feature/HU-AGREGAR-OFICINA-REGISTRAL-BCW
```

## Step 3: Generate and apply changes

### bcv-bacc-party-lifecycle-management-service

**Modified:**

- `BusinessAccountRecord.java` — added `registryOffice` field.
- `CreateBusinessAccountService.java` — added validation and persistence.
- `SplValidationMapper.java` — mapped `registryOffice` to `oficinaRegistral`.

**Created:**

- `RegistryOfficeEntity.java`
- `RegistryOfficeRepository.java`
- `V202608261830__create_registry_office_table.sql`
- `BusinessAccountRecordTest.java`

### bcv-bacc-channel-activity-service

No changes needed. The field `oficinaRegistral` already exists in the DTO.

## Step 4: Run validations

```bash
./mvnw verify
```

Results:

```text
bcv-bacc-party-lifecycle-management-service:
  Linter: passed
  Tests: 127 passed, 0 failed

bcv-bacc-channel-activity-service:
  Linter: passed
  Tests: 84 passed, 0 failed
```

## Step 5: Implementation report

The skill returns the report with diff summary and next steps.

## Step 6: Manual review

The developer reviews the diff, commits, pushes, and creates pull requests.
