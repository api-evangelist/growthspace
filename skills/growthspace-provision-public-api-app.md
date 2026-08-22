---
name: growthspace-provision-public-api-app
description: >-
  Provision a Growthspace Public API application for a company, grant it scopes
  from the published catalogue, and issue its first bearer token.
api: growthspace-public-api-management
operations:
  - AppsManagementController_getScopes
  - AppsManagementController_createApp
  - AppsManagementController_listApps
  - AppsManagementController_updateScopes
  - AppsManagementController_generateToken
generated: '2026-08-22'
method: generated
source: openapi/growthspace-public-api-management-openapi-original.yml
---

# Provision a Growthspace Public API application

Base URL: `https://public-api-management-dot-growthspace-246311.oa.r.appspot.com`

Growthspace publishes no developer documentation for this service. Every
operationId below is verified present in the OpenAPI the service itself serves
at `/api/docs-json`. The request bodies (`CreateAppDto`, `UpdateScopesDto`) are
declared in that spec as objects with **no properties**, so their fields are not
knowable from the contract — do not guess them. Obtain the body shape from
Growthspace before sending a write.

## Steps

1. **Read the scope catalogue.**
   `AppsManagementController_getScopes` — `GET /admin/scopes`.
   Returns `{"data": [...]}` with 11 scopes. This endpoint answers without
   credentials. Choose the narrowest set that covers the job:
   - read only: `programs.read`, `participants.read`, `workshops.read`,
     `company.read`, `reporting.read`, `integration.read`
   - write: `programs.write`, `participants.write`, `workshops.write`,
     `integration.write`
   - never request `joker` — it is global cross-company access, GS Admin only.
   `reporting` has no write variant by design.

2. **Check what already exists for the company.**
   `AppsManagementController_listApps` — `GET /admin/apps?companyId=<id>`.
   `companyId` is required. Reuse an existing app rather than creating a
   duplicate: there is no de-duplication contract on create (see step 3).

3. **Create the application.**
   `AppsManagementController_createApp` — `POST /admin/apps` with a
   `CreateAppDto` body. Returns 201.
   **This call is not idempotent.** The API declares no `Idempotency-Key`
   parameter anywhere, so a retried or duplicated request creates a second
   application. Confirm with step 2 before retrying a request whose response you
   did not see.

4. **Set the granted scopes.**
   `AppsManagementController_updateScopes` — `PUT /admin/apps/{appId}/scopes`
   with an `UpdateScopesDto` body.
   This is a **replace**, not a merge, and there is no restore operation. Read
   the current scope set first and keep a copy — the only way back to the
   previous grant is to send it again yourself.

5. **Issue the token.**
   `AppsManagementController_generateToken` — `POST /admin/apps/{appId}/token`.
   Returns 201 with the application credentials. Store the `clientSecret` at
   once; the admin console presents it as a copy-once value.

## Error handling

Errors come back as `{"msg": "..."}` with `content-type: application/json` — not
RFC 9457 problem+json. There is no error code, no type URI and no request id, so
branch on the HTTP status, not on the message. The spec declares **no** 4xx or
5xx responses for any operation, so treat any non-2xx as opaque and surface the
raw status to the caller.

## Limits and safety

- No rate-limit headers are returned and no limits are published. Back off on
  your own schedule.
- No dry-run or preview mode exists.
- Do not chain step 3 into step 4 and 5 unattended on a production tenant: an
  application created in error can only be removed by
  `AppsManagementController_revokeApp`, which Growthspace states cannot be
  undone.
