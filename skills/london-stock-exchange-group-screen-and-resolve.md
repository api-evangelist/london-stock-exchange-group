---
name: Screen an entity and resolve the results
description: Create a World-Check One screening case for an individual or organisation, retrieve the matches, and review/resolve them using the group's resolution toolkit.
api: openapi/london-stock-exchange-group-case-api-openapi.yml
operations: [getGroups, getCaseTemplateForGroup, getResolutionToolkitForGroup, saveCase, screenCase, getCase, getResults, reviewResults, resolveResults]
generated: '2026-06-20'
method: generated
---

# Screen an entity and resolve the results

Base URL: `https://api-worldcheck.refinitiv.com/v2` (API v2). Every request must carry
an HMAC-SHA256 `Authorization: Signature ...` header and a current RFC 1123 `Date`
header, with `Content-Type: application/json` (see
`authentication/london-stock-exchange-group-authentication.yml`). API v3 lives at
`https://api.risk.lseg.com/screening/v3/` with OAuth 2.0 service accounts.

## Steps

1. **Verify credentials and find your group** — call `getGroups` (`GET /groups`).
   A `200` with your accessible groups confirms auth. Note the `groupId` you will
   screen under; `401` means the signature or `Date` header is wrong.
2. **Get the case template** — `getCaseTemplateForGroup`
   (`GET /groups/{groupId}/caseTemplate`) to learn which secondary fields
   (date of birth, country) and custom fields the group requires.
3. **Create the case** — `saveCase` (`POST /cases`) with the entity name, type
   (INDIVIDUAL / ORGANISATION / VESSEL), provider types, and any secondary fields
   from the template. Record the returned `caseSystemId`.
4. **Screen it** — `screenCase` (`POST /cases/{caseSystemId}/screeningRequest`).
   For one-shot screening without persisting a case you can instead use `screen`
   (`POST /cases/screeningRequest`).
5. **Fetch the matches** — `getResults` (`GET /cases/{caseSystemId}/results`).
   Each result carries a `resultId` and match metadata against World-Check
   profiles.
6. **Review and resolve** — load the group's resolution options with
   `getResolutionToolkitForGroup` (`GET /groups/{groupId}/resolutionToolkit`),
   then `reviewResults` (`PUT /cases/{caseSystemId}/results/review`) to add
   remarks and `resolveResults` (`PUT /cases/{caseSystemId}/results/resolution`)
   with a valid status/risk/reason combination from the toolkit.

## Rules

- There is no idempotency contract: retrying `saveCase` after a timeout can create
  duplicate cases — check with `getCaseReference` (`GET /caseReferences`) first.
- Errors return a custom `Error` envelope (`error`, `cause`, `objectId`), not
  RFC 9457 — see `errors/london-stock-exchange-group-problem-types.yml`.
- Every operation can return `429`; back off and retry (no `Retry-After` header).
- `415` means you forgot `Content-Type: application/json`.
