# Startup Troubleshooting for BCV Spring Boot

Use this checklist for local or environment startup failures.

## Quick triage

1. Confirm active profile (`local`, `dev`, etc.).
2. Validate Java version required by the project.
3. Verify Config Server and Key Vault reachability.
4. Check unresolved placeholders in logs.
5. Validate DB/broker host and credentials source.

## Frequent failure patterns

| Symptom | Likely cause | Recommended action |
| --- | --- | --- |
| App stops during bootstrap | Config Server unavailable | Enable local fallback profile and verify bootstrap endpoints |
| Placeholder resolution errors | Missing env vars or secrets | Validate required env vars and secret keys |
| Bean creation failure in infra | Dependency missing/unreachable | Apply mock-first for local development, isolate integration beans |
| Port already in use | Local conflict | Change server port for local profile |
| Circular dependency | Wiring issue in app module | Refactor bean graph and separate responsibilities |

## Mock-first local strategy

When external systems are unavailable:

1. Keep HTTP/client contracts explicit.
2. Add stub adapters for local profile.
3. Validate request/response contracts with tests.
4. Switch to real adapters only after connectivity and contract checks pass.

## Minimal verification after fix

- Application starts successfully on expected profile.
- Health endpoints respond.
- No secret values printed in logs.
- Key flows pass smoke tests.
