# Regulatory Compliance

Country-specific regulatory docs required to rent numbers (India, UK, Germany, etc.). Five interrelated resources:

- **ComplianceApplication** — bundles docs for a country+type+user-type combo (lifecycle entity)
- **ComplianceRequirement** — what docs are needed (varies by country/type/user-type)
- **ComplianceDocument** — individual docs uploaded
- **EndUser** — business or individual identity tied to applications
- **Profile** — account-level profile context

Docs: https://www.plivo.com/docs/numbers/compliance, /numbers/regulatory-compliance

## End-to-end workflow

1. Query **ComplianceRequirement** for needed docs (country + type + user-type)
2. Create **EndUser** (business or individual identity)
3. Upload **ComplianceDocument** files
4. Create **ComplianceApplication** linking the EndUser + Documents
5. Wait for approval (`pending` → `approved`/`rejected`)
6. Approved app can be assigned to multiple numbers in the same country+type

## Compliance Applications

Real path is `/PhoneNumber/Compliance/`, NOT `/Compliance/Application/`.

| Method | Path | Purpose |
|---|---|---|
| POST | /PhoneNumber/Compliance/ | Create — Tier 1 |
| GET | /PhoneNumber/Compliance/ | List (key: `compliances[]`) |
| GET | /PhoneNumber/Compliance/{compliance_id}/ | Get |
| POST | /PhoneNumber/Compliance/{compliance_id}/ | Update |
| DELETE | /PhoneNumber/Compliance/{compliance_id}/ | Delete |

## Compliance Requirements (read-only)

| Method | Path |
|---|---|
| GET | /ComplianceRequirement/?end_user_type=X&country_iso2=X&operation_type=X&phone_number=X |

REQUIRED query params: `end_user_type` (e.g. `business` / `individual`), `country_iso2`, `operation_type`, `phone_number`. Missing params return HTTP 400 with `acceptable_document_types` array in the error payload.

Returns list of doc types needed for the requested combination.

## Compliance Documents

| Method | Path | Purpose |
|---|---|---|
| POST | /ComplianceDocument/ | Upload — Tier 1 |
| GET | /ComplianceDocument/ | List (key: `objects[]`) |
| GET | /ComplianceDocument/{doc_id}/ | Get |
| POST | /ComplianceDocument/{doc_id}/ | Update |
| DELETE | /ComplianceDocument/{doc_id}/ | Delete |

Upload includes the document type (passport, business reg, etc.) and the file.

## End User (Identity)

| Method | Path | Purpose |
|---|---|---|
| POST | /EndUser/ | Create — Tier 1 |
| GET | /EndUser/ | List (key: `objects[]`) |
| GET | /EndUser/{end_user_id}/ | Get |
| POST | /EndUser/{end_user_id}/ | Update |
| DELETE | /EndUser/{end_user_id}/ | Delete |

Represents the business or individual that owns/uses the phone number.

## Profile

| Method | Path | Purpose |
|---|---|---|
| GET | /Profile/ | List (key: `profiles[]`) |
| POST | /Profile/ | Create |
| GET | /Profile/{profile_id}/ | Get |

Account/organization context for compliance applications. Less commonly used directly.

## Status

`pending` → `approved` / `rejected`. Country-specific; can take days to weeks.

## Gotchas

- Document upload often easier via Console UI than via API
- Approval is regulator-side — Plivo can't fast-track
- Without compliance, certain country numbers can't be rented
- Compliance can expire (e.g., India address proof valid for fixed period); re-submit if expiring
- `ComplianceRequirement` is your contract — query it first, don't upload docs blind
- `/ComplianceApplication/` (legacy) also returns 200 — prefer `/PhoneNumber/Compliance/` per current docs
- All compliance ops use `objects[]` shape EXCEPT:
  - `/PhoneNumber/Compliance/` uses `compliances[]`
  - `/Profile/` uses `profiles[]`
