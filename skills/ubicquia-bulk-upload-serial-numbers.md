---
name: Bulk-load device serial numbers from a production file
description: Upload a production CSV of Ubicquia device serial numbers, poll ingestion status, and verify the resulting serial records.
api: openapi/ubicquia-config-api-openapi-original.json
operations:
  - update
  - check-uploaded-production-file-status
  - get-production-file-list
  - get-serial-number-list
  - destroy
---

# Bulk-load Ubicquia device serial numbers

Base URL: `https://config.api.ubicquia.com/api/` · Auth: `x-api-key` header.

Device identity in this API is a serial-number row carrying the full radio identity set:
`serial_number`, `dev_eui`, `iccid`, `imei`, `imsi`, `mac_address`, `sku_part_number`,
`firmware_version`, `box_id`, `carton_id`, `uid` — joined to a `customer_id` and a
`sales_order_number`.

## Step 1 — upload the production file

`update` — `POST /update-production-file` ("Update Serial Number")

Ingests a CSV. Like every bulk write here it is **async**: the rows are processed in the background.

> Address this operation as `POST /update-production-file`. Its `operationId` is the bare string
> `update`, which is reused by three other operations in this spec.

## Step 2 — poll ingestion

`check-uploaded-production-file-status` — `GET /check-uploaded-production-file-status`

Returns a count-shaped payload in the standard envelope. Poll until ingestion settles before
verifying.

## Step 3 — verify

- `get-production-file-list` — `GET /get-production-file-list` lists the upload records themselves
  (`filename`, `uploaded_filename`, `user_name`, `created_at`, `deleted_at`, `path`), and supports
  `sort_by` / `sort_dir`, `file_type`, `q` + `search_type`, `start_date_time` / `end_date_time`, and
  `page` / `per_page`.
- `get-serial-number-list` — `GET /get-serial-number-list` lists the ingested device rows, filterable
  by `customer_id`, `so_number` and `q` + `search_type`.

**Pagination.** Both are Laravel length-aware paginators: send `page` and `per_page`, then follow
`links.next` until it is null. Read `meta.total` for the count.

## Step 4 — remove a bad upload

`destroy` — `POST /delete-production-file`.

The serial and transformer schemas carry a `deleted_at` field, which suggests rows are soft-deleted
server-side — **but no restore operation is exposed and no retention window is published.** Treat this
call as final and get human confirmation first.
