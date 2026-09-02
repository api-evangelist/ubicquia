---
name: Verify a Ubicquia API key and discover the surface
description: Confirm an x-api-key credential works against the Ubicquia Config API before running any read or write, and understand what the 39-operation surface covers.
api: openapi/ubicquia-config-api-openapi-original.json
operations:
  - index
---

# Verify a Ubicquia API key

Base URL: `https://config.api.ubicquia.com/api/`

## Auth

Every operation in this API takes one credential: the `x-api-key` request header. There is no OAuth,
no bearer token, no refresh and no scopes. The spec states it plainly: *"Use header x-api-key: 'key'
to send api key endpoints"*.

## Step 1 — probe the key

Call `index` (`GET /AuthCheck`). It exists for exactly this purpose.

```
GET https://config.api.ubicquia.com/api/AuthCheck
x-api-key: <your key>
```

- A non-401 response means the key is accepted.
- A `401` returns the vendor envelope `{"status":"failed","code":"401","message":"Unauthenticated"}`.
  There is no distinction between a missing key and a wrong key — both are 401.

## Step 2 — know what you are holding

The surface is a **manufacturing and provisioning** API, not the UbiVu telemetry platform. It is
organized into eight tags:

| Tag | What it covers |
|---|---|
| AuthCheck | key verification |
| Sales Order | create/read/update/delete sales orders, poll async creation |
| Fulfillment Details | per-device fulfillment records and their async jobs |
| Serial Number | bulk production-file (CSV) upload of device serials |
| Transformer Details | bulk transformer and coil file upload |
| ICCID Master | cellular SIM inventory: bulk CSV update/delete, per-ICCID status |
| Customer Detail | resolve a customer id |
| User | user CRUD and customer-status changes |

## Rules that apply to every call in this API

- **Envelope.** Success and failure both return `{data, status, code, message, version}`. `status` is
  the string `"success"` or `"failed"`. There is no machine-stable error code beyond the HTTP status,
  and errors are `application/json`, **not** RFC 9457 `application/problem+json`.
- **No idempotency.** No `Idempotency-Key` exists. Do not blind-retry a `POST` — re-issuing
  `add-sales-order` or `store` (`POST /fulfillment-detail`) creates another record.
- **No rate-limit signal.** No operation declares a `429` and no `RateLimit-*` / `Retry-After` header
  is documented. Pace yourself; you will get no runtime hint.
- **Deletes are final.** Seven destructive operations exist and none has a published restore path or
  retention window. See `conventions/ubicquia-conventions.yml` → `reversibility`.
- **Watch out for the operationIds.** Several are generic and duplicated across tags (`store`,
  `update`, `destroy`, `index`). Always address an operation by **method + path**, not by
  `operationId` alone.
