---
name: Manage the Ubicquia ICCID (cellular SIM) master inventory
description: Bulk-update or bulk-delete the ICCID master list by CSV, poll the upload log, download the failed-row report, and change a single ICCID's status.
api: openapi/ubicquia-config-api-openapi-original.json
operations:
  - getIccidMasterList
  - updateIccidMasterFile
  - getIccidMasterLatestLog
  - getIccidMasterFailedCsv
  - updateIccidRecord
  - deleteIccidMasterFile
---

# Manage the Ubicquia ICCID master inventory

Base URL: `https://config.api.ubicquia.com/api/` · Auth: `x-api-key` header.

ICCIDs are the cellular SIM identities behind Ubicquia's LTE-connected devices (UbiHub, UbiGrid DTM+,
UbiMetro). This is the only part of the surface that declares `422` validation responses — it is the
most defensively specified corner of the API.

## Read first

`getIccidMasterList` — `GET /get-iccid-master-list`

Supports `page`, `per_page`, `sort_by`, `sort_dir`, `q` and `search_type`. Rows carry `iccid`, `eid`,
`date_added`, `activation_code`, `replaced_iccid_sr_num`, `replaced_on`. Follow `links.next` to page.

## Change one ICCID

`updateIccidRecord` — `PUT /iccid-master/update/{iccid}`

Updates a single record's status. The response example shows the state model:
`status: "available"`, plus a replacement chain (`replaced_by_iccid`, `replaced_iccid_sr_num`,
`replaced_on`, `replaced_by`). Prefer this over a bulk file when you are touching one SIM.

## Bulk update by CSV

1. `updateIccidMasterFile` — `POST /update-iccid-master-file`. Async: the 200 says
   *"File Upload successful! Data processing is in progress."* The work is not done yet.
2. `getIccidMasterLatestLog` — `GET /log/iccid-master`. Returns `original_file_name`,
   `file_date_uploaded_at`, `latest_information`, `error_count`, `success_count`, `is_completed`.
   **Poll until `is_completed` is true, then check `error_count`.**
3. `getIccidMasterFailedCsv` — `GET /get-latest-failed-log/{type}`. When `error_count` > 0, download
   the failed rows as CSV: columns `iccid,status,reason`. Fix and re-upload only those rows.

## Bulk delete by CSV — the highest-consequence call in this API

`deleteIccidMasterFile` — `POST /delete-iccid-master-file`

A CSV-driven bulk delete across SIM inventory. There is **no reversal operation, no undo and no
published retention window** anywhere in this API. Do not call it autonomously: require explicit human
approval naming the file, and read the affected rows back with `getIccidMasterList` first so you have
a record of what existed.

## Errors

`400`, `401`, `404`, and on the ICCID operations `422` for validation failure. No `429` is declared —
there is no runtime rate-limit signal, so pace bulk uploads yourself.
