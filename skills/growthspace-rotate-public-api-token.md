---
name: growthspace-rotate-public-api-token
description: >-
  Refresh or rotate the bearer token of an existing Growthspace Public API
  application, and revoke the application when access must end.
api: growthspace-public-api-management
operations:
  - AppsManagementController_listApps
  - AppsManagementController_refreshForApp
  - AppsManagementController_publicRefresh
  - AppsManagementController_generateToken
  - AppsManagementController_revokeApp
generated: '2026-08-22'
method: generated
source: openapi/growthspace-public-api-management-openapi-original.yml
---

# Rotate or revoke a Growthspace Public API credential

Base URL: `https://public-api-management-dot-growthspace-246311.oa.r.appspot.com`

Two refresh paths exist and they are not interchangeable.

## Refresh from the integration side

`AppsManagementController_publicRefresh` — `POST /public/refresh` with a
`RefreshTokenDto` body. This is the endpoint an integration owns: it is the only
route in the contract under `/public/`, and it is how a running client renews its
own credential. Returns 201.

## Refresh from the admin side

`AppsManagementController_refreshForApp` — `POST /admin/apps/{appId}/refresh`
with a `RefreshTokenDto` body. Same body type, but scoped to a specific `appId`
and served from the admin surface. Use this when operating on someone else's
application, not from inside the integration.

## Mint a new token instead of refreshing

`AppsManagementController_generateToken` — `POST /admin/apps/{appId}/token`.
Use when the refresh token itself is lost or suspect.

## Confirm which application you are touching

`AppsManagementController_listApps` — `GET /admin/apps?companyId=<id>`.
`appId` is a path parameter on every mutating call above and the API gives no
confirmation prompt. Resolve the id from this list before acting.

## Revoke — irreversible

`AppsManagementController_revokeApp` — `DELETE /admin/apps/{appId}`.
Growthspace's own console warns that revoking an application **cannot be
undone**. There is no restore operation, no soft-delete and no stated grace
window anywhere in the contract or the product. Never issue this call
autonomously; require explicit human confirmation of the `appId` first.

## Conventions that apply to all of the above

- No `Idempotency-Key` is supported. A retried refresh is a second refresh.
- Errors return `{"msg": "..."}` with no code and no request id; branch on the
  HTTP status.
- No rate-limit headers are returned.
- `RefreshTokenDto` is declared in the spec with no properties; get the field
  names from Growthspace rather than assuming them.
