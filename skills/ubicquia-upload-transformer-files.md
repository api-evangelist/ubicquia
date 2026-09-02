---
name: Load distribution transformer and coil files
description: Upload transformer or coil CSV files to the Ubicquia Config API, poll ingestion, list the uploads and fetch one back by id.
api: openapi/ubicquia-config-api-openapi-original.json
operations:
  - update
  - update-coil
  - check-uploaded-file-status
  - get-transformer-file-list
  - get-transformer-file-by-id
  - get-coil-file-by-id
  - destroy-coil
---

# Load Ubicquia transformer and coil files

Base URL: `https://config.api.ubicquia.com/api/` · Auth: `x-api-key` header.

This is the provisioning side of Ubicquia's grid business — the distribution transformer records that
UbiGrid DTM+ monitoring attaches to.

## Step 1 — upload

- Transformers: `update` — `POST /update-transformer-file` ("Update transformer details")
- Coils: `update-coil` — `POST /update-coil-file` ("Update coil transformer details")

Both ingest a CSV asynchronously. Address them by **method + path**; `update` as an `operationId` is
reused elsewhere in this spec.

## Step 2 — poll

`check-uploaded-file-status` — `GET /check-uploaded-file-status`. Returns a count payload in the
standard `{data, status, code, message, version}` envelope. Wait for it to settle.

## Step 3 — list and inspect

- `get-transformer-file-list` — `GET /get-transformer-file-list`. Paginated (`page`, `per_page`),
  sortable (`sort_by`, `sort_dir`), searchable (`q` + `search_type`) and date-windowed
  (`start_date_time`, `end_date_time`).
- `get-transformer-file-by-id` — `GET /get-transformer-file-by-id/{id}` returns
  `{url, fileExtension}` — a link to the stored file rather than its contents.
- `get-coil-file-by-id` — `GET /get-coil-file-by-id/{id}` does the same for a coil file.

## Step 4 — remove

`destroy` — `POST /delete-transformer-file`, and `destroy-coil` — `POST /delete-coil-file`.

`TransformerDetail` declares a `deleted_at` field, so rows are likely soft-deleted server-side, **but
no restore operation is published and no retention window is stated**. Treat both deletes as final and
confirm with a human first.

## Errors

`400`, `401`, `404`. No `429`, no `5xx`, and no `Retry-After`. See
`errors/ubicquia-problem-types.yml`.
