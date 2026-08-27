# Example: "Add registry office in BCW channel"

## Input HU

> Note: the skill accepts HUs in any language. This example uses an English translation of a real BCV story.

```text
As an SPL operator in the BCW channel
I want to select the registry office from a predefined list
so that the registration is correct and manual errors are avoided.

Acceptance criteria:
CA1: ...
CA2: ...

Workspace: /Users/joseluis/Downloads/bcv-bacc-account-opening-reporting-service
```

## Commands executed

```bash
# Validate HU → OK

# Classify services from READMEs
# - party-lifecycle-management-service → primary
# - channel-activity-service → secondary
# - service-point-service → to_confirm
# - account-opening-reporting-service → to_confirm

# Build graphs if missing
graphify bcv-bacc-party-lifecycle-management-service --code-only
graphify bcv-bacc-channel-activity-service --code-only

# Query 1 (primary): does the concept exist?
graphify query "registry office" --max 5 \
  --graph bcv-bacc-party-lifecycle-management-service/graphify-out/graph.json
# Result: no matching nodes → new concept

# Query 2 (primary): find SPL validation flow
graphify query "PowersValidation" --max 5 --context_filter call \
  --graph bcv-bacc-party-lifecycle-management-service/graphify-out/graph.json
# Result: PowersValidationService, SplValidationMapper, PowersValidationPublisher

# Query 3 (secondary): does channel-activity already support the field?
graphify query "oficinaRegistral" --max 5 \
  --graph bcv-bacc-channel-activity-service/graphify-out/graph.json
# Result: PowersValidationMessageInDto already has oficinaRegistral

# Targeted reads (3/3)
read SplValidationMapper.java offset=90 limit=80
read PowersValidationOutDto.java limit=80
read ExpedientEntity.java limit=80

# Generate context
# .context/hu-add-registry-office-in-bcw-channel.md
```

## Generated context (excerpt)

```markdown
# Context: Add registry office in BCW channel

## 1. Metadata

| Field | Value |
| --- | --- |
| Code | `add-registry-office-in-bcw-channel` |
| Actor | SPL operator in BCW channel |
| Action | Select registry office from a predefined list |
| Goal | Ensure correct registration and avoid manual errors |

## 4. Services

### 4.1 Primary

| Service | Role | Injection point |
| --- | --- | --- |
| `bcv-bacc-party-lifecycle-management-service` | Receives request, persists expedient, sends SPL request | `SplValidationMapper.toSplRequestValidationDomain` |

### 4.2 Secondary

| Service | Role | Impact |
| --- | --- | --- |
| `bcv-bacc-channel-activity-service` | Forwards SPL message to cross Service Bus | Must preserve `oficinaRegistral` |

### 4.4 To Confirm

| Service | Reason |
| --- | --- |
| `bcv-bacc-service-point-service` | May need to display/edit in BCW service tray |
| `bcv-bacc-account-opening-reporting-service` | May need to report field in Teradata/commercial queries |

## 6. Key Finding

`registryOffice` already exists in `PowersValidationOutDto` mapped to JSON `oficinaRegistral`,
but it is currently computed from `legalEntityDomain.address.province` instead of being a controlled catalog value.

## 10. Gaps

| ID | Topic | Reason | Impact | Blocking |
| --- | --- | --- | --- | --- |
| G1 | Catalog source | No master catalog service or parametric table found | Blocks CA6 (data integrity) | No |
| G2 | service-point impact | Unclear if BCW tray must show/edit the field | UI/UX scope | No |
| G3 | reporting impact | Unclear if Teradata/commercial reports must include the field | Reporting scope | No |
```

## Response to user

```text
Context generated: .context/hu-add-registry-office-in-bcw-channel.md

Services:
- Primary: bcv-bacc-party-lifecycle-management-service
- Secondary: bcv-bacc-channel-activity-service
- To confirm: bcv-bacc-service-point-service, bcv-bacc-account-opening-reporting-service

Main injection point: SplValidationMapper.toSplRequestValidationDomain

Key finding: registryOffice already exists in PowersValidationOutDto but is computed from address.province.

Top 3 gaps:
1. No catalog source found (blocks data integrity validation).
2. service-point-service impact unclear.
3. account-opening-reporting-service impact unclear.

Next step: bcv-dhu-writer .context/hu-add-registry-office-in-bcw-channel.md
```
