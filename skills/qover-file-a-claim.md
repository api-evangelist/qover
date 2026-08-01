---
name: File and track a Qover insurance claim
description: File an embedded-insurance claim through the Qover Claims API, attach supporting documents, and track its status to resolution.
api: https://docs.qover.com/
operations:
  - GET /claims/claim-schemas/{product}/{country}
  - POST /dam/v1/assets
  - POST /claims/v1/claims
  - GET /claims/v1/claims/{claimId}
  - GET /claims/v1/claims/{claimId}/statuses
  - GET /claims/v1/claims/{claimId}.pdf
---

# File and track a Qover insurance claim

Use this skill to file a claim on behalf of a customer through Qover's Claims API and follow it to resolution. All requests are JSON over HTTPS against the Qover API host (sandbox: `https://api.sbx.qover.io`).

## Authentication

Every request must include your Qover **API key in the request header**. Qover issues a Sandbox key first and a Production key after review. Never hard-code or commit the key. See `authentication/qover-authentication.yml`.

## Steps

1. **Get the validation schema for the product/country.**
   `GET /claims/claim-schemas/{product}/{country}` (e.g. product `FINTECHX`, country `BE`). Use the returned schema to build a valid claim body — required fields depend on the per-partner product configuration.

2. **(If there are attachments) create the document assets first.**
   `POST /dam/v1/assets` to register each attachment; this returns a signed `uploadURL`. `PUT` the file bytes to that URL, then reference the asset id when creating the claim.

3. **Create the claim.**
   `POST /claims/v1/claims` with a body that satisfies the schema from step 1 (claimant/customer, policy reference, incident details, and any asset references). The response contains the claim `id` and `claimNumber`.

4. **Retrieve the claim.**
   `GET /claims/v1/claims/{claimId}` to read the current state, including `status` (`code`, `category`, `name`) and `_links`.

5. **Track status history.**
   `GET /claims/v1/claims/{claimId}/statuses` for the full status timeline (e.g. eligibility checks, declined, completed). Alternatively, configure a **webhook** so Qover pushes `claim.status.changed` events to your endpoint (see `asyncapi/qover-webhooks.yml`).

6. **Download the claim report.**
   `GET /claims/v1/claims/{claimId}.pdf` to obtain the PDF claim report.

## Conventions & rules

- **Versioning:** URI path, `v1`.
- **Pagination:** list endpoints accept `page`, `size`, `sort`, `sortDirection` (see `conventions/qover-conventions.yml`).
- **Hypermedia:** resources carry HAL-style `_links` with `self.href`.
- **Product context:** claim shape is governed by `product` + `country`; always fetch the schema before building a body.
- **Idempotency:** no idempotency-key mechanism is documented — do not assume safe automatic retries on `POST /claims/v1/claims`.
