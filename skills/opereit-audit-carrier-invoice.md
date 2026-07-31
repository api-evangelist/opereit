---
name: Audit a carrier invoice against a contract
description: Upload a carrier contract, upload a carrier invoice, run an audit against the contract, and retrieve the discrepancy findings.
api: openapi/opereit-invoice-auditing-openapi.json
operations: [uploadContract, createInvoiceAudit, listInvoiceAudits, listInvoiceAuditFindings]
---

# Audit a carrier invoice against a contract

Use the Opereit Invoice Auditing API to detect over-billing on a carrier
invoice. All operations below exist in `openapi/opereit-invoice-auditing-openapi.json`.

## Auth
HTTP Basic Auth with an API key pair. Send `Authorization: Basic base64(key_id:key_secret)`;
the secret is prefixed `opr_live_`. Base URL: `https://api.opereit.com`.

## Steps
1. **Upload the contract** — `uploadContract` (`POST /v1/contracts`), multipart
   with `file`, `carrier_name`, `signing_date`. The contract is created with
   `status=PROCESSING`.
2. **Wait for extraction** — poll `listContracts` (`GET /v1/contracts?status=ACTIVE`)
   until the contract's `status=ACTIVE` (rates and surcharges are then extracted).
   If `status=FAILED`, read `status_reason`.
3. **Create the audit** — `createInvoiceAudit` (`POST /v1/invoice-audits`),
   multipart with `file` (the carrier invoice, PDF or CSV) and `contract_id`.
   Returns `status=PENDING`.
4. **Wait for the audit** — poll `listInvoiceAudits`
   (`GET /v1/invoice-audits?contract_id=...`) until `status=COMPLETE`
   (transitions PENDING → INGESTING → AUDITING → COMPLETE; or FAILED).
5. **Read the findings** — `listInvoiceAuditFindings`
   (`GET /v1/invoice-audits/{audit_id}/findings`). Each finding has a `type`
   from the catalog: RATE_MISMATCH, RATE_NOT_FOUND, SURCHARGE_MISMATCH,
   SURCHARGE_NOT_FOUND, WAIVED_SURCHARGE_BILLED, DUPLICATE_CHARGE, plus
   `expected_value` vs `actual_value` and a `confidence`.

## Conventions
- Cursor pagination: `?limit=50&cursor=<opaque>`; response is
  `{ data, pagination: { cursor, has_more } }`. Follow `pagination.cursor`
  while `has_more` is true.
- Amounts/weights/rates are decimal **strings**; fields are `snake_case`.
- Errors return `{ "error": "..." }` (401 without valid auth, 400 on bad input,
  404 if the contract/audit id is unknown).
- There is no idempotency-key header; do not blindly retry a POST that may have
  succeeded — list first to check for a duplicate.
