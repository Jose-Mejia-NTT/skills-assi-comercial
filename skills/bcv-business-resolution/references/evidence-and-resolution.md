# Evidence and resolution policy

## Purpose

Use this reference to rank business-resolution evidence before handing candidates to technical impact analysis.

## Evidence hierarchy

Prefer sources in this order:

1. Current business capability catalog with owner and verification metadata.
2. Approved functional HU/HAB or domain decision.
3. Current service documentation and service-context reports.
4. Current source code or architecture graph artifacts, used for technical confirmation in Step 2 only.
5. README or draft snapshot documents.
6. Name similarity or generic entity matching.

This skill uses levels 1-3 to produce business candidates. The next technical phase must use levels 4-5 to confirm implementation details. Level 6 is never sufficient by itself.

## Candidate roles

- `owner`: service responsible for the business capability.
- `orchestrator`: service coordinating the business flow without necessarily owning the data.
- `data_owner`: service responsible for the authoritative business state.
- `participant`: service involved in the flow.
- `producer`: service publishing a business event or message.
- `consumer`: service consuming a business event or message.
- `integration_adapter`: service that translates an external system interaction.
- `possible_downstream`: likely recipient of a derived report or notification.
- `unknown`: no supported ownership decision.

## Confidence rules

- `high`: explicit capability ownership supported by an approved catalog or functional decision.
- `medium`: strong domain relationship supported by multiple sources, but ownership or contract is not confirmed.
- `low`: plausible candidate based on incomplete or draft evidence; always requires review.

Never convert confidence into an unexplained percentage.

## Required discrepancy handling

Mark `REVIEW_REQUIRED` when:

- a draft or snapshot document is the main evidence;
- two services could own the same capability;
- the request mixes orchestration, data ownership and downstream reporting;
- a required external system is not present in the available repository set;
- the catalog and functional request disagree.

Mark `BLOCKED` when no candidate can be supported without inventing a mapping.

## BACC pilot facts

The current BACC pilot documentation supports these starting points:

- `bcv-bacc-party-lifecycle-management-service` is the opening-flow orchestrator.
- `bcv-bacc-customer-service` is the customer and legal-entity data owner candidate.
- `bcv-bacc-account-opening-reporting-service` is the Teradata reporting downstream candidate.
- The sector-manager ownership and CICS path are not confirmed by the available service documentation alone.
- The external `bcv-bacc-expedients-service` is referenced as a critical dependency but is outside the local repository set.

These are candidates for handoff, not implementation decisions.
