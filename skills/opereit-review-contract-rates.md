---
name: Review extracted contract rates and surcharges
description: Upload a carrier contract and review the rate-card entries and surcharges Opereit extracts from it.
api: openapi/opereit-invoice-auditing-openapi.json
operations: [uploadContract, listContracts, listContractRates, listContractSurcharges]
---

# Review extracted contract rates and surcharges

Verify what Opereit extracted from a carrier contract before auditing invoices
against it. All operations exist in `openapi/opereit-invoice-auditing-openapi.json`.

## Auth
HTTP Basic Auth (`Authorization: Basic base64(key_id:key_secret)`), key secret
prefixed `opr_live_`. Base URL `https://api.opereit.com`.

## Steps
1. **Upload the contract** — `uploadContract` (`POST /v1/contracts`) with `file`,
   `carrier_name`, `signing_date` (optional `contract_number`, `account_number`).
2. **Confirm it is active** — `listContracts` (`GET /v1/contracts`); wait until
   the target contract shows `status=ACTIVE`, and note `rate_count` /
   `surcharge_count`.
3. **List the rates** — `listContractRates`
   (`GET /v1/contracts/{contract_id}/rates`). Filter with `origin_country`,
   `destination_country`, `service_type`, `weight_min`, `weight_max`. Each rate
   has `weight_mode` (FIXED|RANGE) and `rate_type` (FLAT|PER_KG).
4. **List the surcharges** — `listContractSurcharges`
   (`GET /v1/contracts/{contract_id}/surcharges`). Filter with `search`,
   `billing_type` (PERCENTAGE|PERCENTAGE_OF_BASE|FIXED|TIERED|PERCENTAGE_OFF),
   `service_type`, `is_waived`. Waived surcharges billed later become
   WAIVED_SURCHARGE_BILLED findings.

## Conventions
- Cursor pagination (`limit`/`cursor`), `snake_case` fields, decimal-string
  amounts, ISO 3166-1 alpha-2 country codes and ISO 4217 currencies.
- Errors: `{ "error": "..." }`; 404 if `contract_id` is unknown.
