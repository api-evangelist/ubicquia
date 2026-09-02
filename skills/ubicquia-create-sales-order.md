---
name: Create a Ubicquia sales order and confirm it landed
description: Submit a sales order to the Ubicquia Config API, poll the async job to completion, and read the resulting order back.
api: openapi/ubicquia-config-api-openapi-original.json
operations:
  - getcustid
  - add-sales-order
  - add-sales-order-config
  - update
---

# Create a Ubicquia sales order

Base URL: `https://config.api.ubicquia.com/api/` · Auth: `x-api-key` header on every call.

## Step 1 — resolve the customer

`getcustid` — `POST /get-customer-id`

Sales orders key on an opaque string `customer_id` (plus a numeric `parent_customer_id` for
hierarchy). Resolve it first rather than assuming one.

## Step 2 — submit the order

`add-sales-order` — `POST /add-sales-order` ("Add Sales Order by async")

This is **asynchronous**. It returns immediately with a job handle; the order is not created yet.

> There is no `Idempotency-Key` on this operation. If the call times out, **do not retry blindly** —
> poll step 3 or read step 4 first, or you will create a duplicate order.

There is also `add-sales-order-config` — `POST /add-sales-order-config` ("Add Sales Order on Config"),
the config-side variant. Pick one; do not fire both for the same order.

## Step 3 — poll to completion

`GET /get-sales-order-status?id=…&customer_id=…` (this operation declares **no** `operationId` in the spec — address it by path)

Poll until the job reports completion. This submit-then-poll shape is the house pattern across this
API (fulfillment, production files, transformer files and ICCID uploads all work the same way).

## Step 4 — read the order back

`GET /get-sales-order/{sales_order_number}` returns the order plus the joined customer record —
including that customer's `ubivu_address`, `api_address` and `mqtt_address`.

## Step 5 — corrections

- `update` — `PUT /update-sales-order` amends an order by id.
- `destroy` — `DELETE /delete-sales-order` removes it. **This is not reversible**: no restore
  operation and no retention window is published. Confirm with a human before calling it.

## Errors

`400` Bad Request · `401` Unauthenticated · `404` Item not found (`{"status":"failed","code":404,
"message":"Operation failed"}`). No `429` and no `5xx` are declared. See
`errors/ubicquia-problem-types.yml`.
