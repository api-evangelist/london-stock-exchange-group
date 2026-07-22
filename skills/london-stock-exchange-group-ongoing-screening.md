---
name: Enable ongoing screening and poll for updates
description: Flag World-Check One cases for ongoing screening (OGS) and periodically retrieve new or changed matches.
api: openapi/london-stock-exchange-group-case-api-openapi.yml
operations: [enableOngoingScreening, disableOngoingScreening, toggleOngoingScreeningByProviderTypes, getOngoingScreeningUpdates, getResults]
generated: '2026-06-20'
method: generated
---

# Enable ongoing screening and poll for updates

Ongoing screening (OGS) re-screens flagged cases against World-Check database
updates so new risk appears without re-submitting the case. Delivery is by
polling — the API publishes no webhooks.

## Steps

1. **Enable OGS on a case** — `enableOngoingScreening`
   (`PUT /cases/{caseSystemId}/ongoingScreening`). For many cases at once use
   `toggleOngoingScreeningByProviderTypes` (`POST /cases/bulk/ongoingScreening`).
2. **Poll for updates** — `getOngoingScreeningUpdates`
   (`POST /cases/ongoingScreeningUpdates`) on a schedule, passing your query
   window. This returns cases whose screening state changed since the last poll.
3. **Pull the changed results** — for each updated case call `getResults`
   (`GET /cases/{caseSystemId}/results`) and route new/changed matches into your
   review workflow.
4. **Disable when off-boarded** — `disableOngoingScreening`
   (`DELETE /cases/{caseSystemId}/ongoingScreening`) when the entity no longer
   needs monitoring.

## Rules

- Poll windows should overlap slightly; the API is rate limited (429 on every
  operation) so keep polling frequency within your provisioned throughput
  (`rate-limits/london-stock-exchange-group-rate-limits.yml`).
- Bulk endpoints are not idempotent — do not blind-retry
  `POST /cases/bulk/ongoingScreening`.
- Use the Pilot account for testing; credentials differ per environment
  (`sandbox/london-stock-exchange-group-sandbox.yml`).
