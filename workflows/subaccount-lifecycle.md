# Workflow: Subaccount Lifecycle (Multi-tenant SaaS)

End-to-end multi-tenant setup using Plivo subaccounts.

## Pre-reqs

- Main account auth_id + auth_token

## Step 1 — Create the subaccount

```bash
curl -X POST "https://api.plivo.com/v1/Account/$MAIN_AUTH_ID/Subaccount/" \
  -u "$MAIN_AUTH_ID:$MAIN_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"tenant-acme-prod","enabled":true}'
```
Returns 201 with `auth_id` (e.g. `SAXXXX...`) + `auth_token`. Persist both in your tenant table.

## Step 2 — Provision tenant resources

**Buy a number, assign to subaccount** (charge hits main):
```bash
# Buy
POST /v1/Account/$MAIN_AUTH_ID/PhoneNumber/{number}/    -d '{}'

# Assign to sub (note: field is `subaccount`, no underscore)
POST /v1/Account/$MAIN_AUTH_ID/Number/{number}/         -d '{"subaccount":"'$SUB_AUTH_ID'"}'
```

**Create a tenant-scoped application** (note: field is `sub_account`, WITH underscore — inconsistency!):
```bash
POST /v1/Account/$MAIN_AUTH_ID/Application/
{
  "app_name":"acme-handler",
  "answer_url":"https://your.app/acme/answer",
  "message_url":"https://your.app/acme/sms",
  "sub_account":"$SUB_AUTH_ID"
}
```

## Step 3 — Hand off tenant credentials

Tenant uses their own auth_id+token:
```bash
curl -u "$SUB_AUTH_ID:$SUB_AUTH_TOKEN" \
  https://api.plivo.com/v1/Account/$SUB_AUTH_ID/Message/
```

Tenant can send SMS, make calls, manage their own resources. Tenant cannot:
- See main's resources or other subaccounts
- Modify their own balance (charges always main)
- See sibling tenants

## Step 4 — Observe usage (from main)

```bash
# Tenant's messages
GET /v1/Account/$MAIN_AUTH_ID/Message/?sub_account=$SUB_AUTH_ID&limit=100

# Tenant's call CDRs
GET /v1/Account/$MAIN_AUTH_ID/Call/?sub_account=$SUB_AUTH_ID&limit=100

# Tenant's recordings
GET /v1/Account/$MAIN_AUTH_ID/Recording/?sub_account=$SUB_AUTH_ID
```

For billing reports: aggregate `TotalAmount` from each tenant's CDRs.

## Step 5 — Pause / suspend a tenant

```bash
POST /v1/Account/$MAIN_AUTH_ID/Subaccount/$SUB_AUTH_ID/
{"enabled":false}
```
Tenant's API calls fail with 403. Existing resources frozen. Reversible — re-enable with `{"enabled":true}`.

## Step 6 — Decommission

```bash
# Option A — release tenant's numbers entirely
DELETE /v1/Account/$MAIN_AUTH_ID/Number/{number}/

# Option B — return them to main pool
POST   /v1/Account/$MAIN_AUTH_ID/Number/{number}/  -d '{"subaccount":""}'

# Then delete the subaccount
DELETE /v1/Account/$MAIN_AUTH_ID/Subaccount/$SUB_AUTH_ID/
```

Subaccount delete is permanent (HTTP 204). Numbers move back to main (unless you released first). Apps/endpoints tied to sub are deleted with it.

## Common pattern: SaaS tenant provisioning

For each new SaaS customer:
1. POST /Subaccount/ → save auth_id, auth_token in your DB
2. POST /PhoneNumber/{n}/ (buy a dedicated number) — Tier 2 cost: ~$0.50/mo
3. POST /Number/{n}/ (assign to subaccount)
4. POST /Application/ with answer_url pointing to `/tenant/$SUB_AUTH_ID/...`
5. POST /Number/{n}/ (attach the application)
6. Store tenant ↔ Plivo mapping in YOUR DB
7. Tenant invoices in YOUR billing system (Plivo's main account = aggregate cost)

## Known limitations to design around

- No per-subaccount budget cap (see not-supported.md) → enforce on your side with CDR polling
- No subaccount auth token rotation API → if leaked, rotate via Console manually
- Field name inconsistency `subaccount` vs `sub_account` — verify per endpoint
- Charges always hit main — design billing/usage tracking yourself
- Disabling subaccount doesn't stop already-in-flight calls/messages; only blocks new
- Deletion is immediate and irreversible — consider disabling first as soft-pause
