---
name: Generate and download case reports
description: Submit an asynchronous World-Check One case report, track its status, and download the finished document.
api: openapi/london-stock-exchange-group-reporting-api-openapi.yml
also: openapi/london-stock-exchange-group-upcoming-api-openapi.yml
operations: [submitRequestForAsyncReport, getSingleReportStatusByReportId, getPaginatedMultipleReportStatus, getReportDataByReportId, getReportErrorDetails, cancelReportByReportId]
generated: '2026-06-20'
method: generated
---

# Generate and download case reports

Report generation is asynchronous: submit, poll status, then download.

## Steps

1. **Submit** — `submitRequestForAsyncReport` (`POST /reports`) with the case
   scope and report options. Record the returned `reportId`.
2. **Poll status** — `getSingleReportStatusByReportId`
   (`GET /reports/{reportId}/status`), or list all active report requests with
   `getPaginatedMultipleReportStatus` (`GET /reports/status`). Poll with backoff
   until the status is complete.
3. **Download** — `getReportDataByReportId` (`GET /reports/{reportId}`) to fetch
   the completed report document.
4. **On failure** — `getReportErrorDetails` (`GET /reports/{reportId}/error`)
   returns a `ReportErrorResponse` (`errorDetails[]` of `objectId`,
   `objectType`, `errorDescription`).
5. **Cancel if no longer needed** — `cancelReportByReportId`
   (`DELETE /reports/{reportId}`).

## Rules

- Submitting the same report request twice creates two reports (no idempotency
  key) — track your `reportId`s.
- All operations require HMAC-signed requests with a current `Date` header;
  `401` usually means clock skew or a stale `Date`.
- `429` applies to polling too — poll status at a modest interval.
