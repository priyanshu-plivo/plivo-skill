# 10DLC (US A2P Messaging Compliance)

Required for SMS from US local long codes to consumers. Three-step:
1. Register a Brand (your company)
2. Register a Campaign (use case)
3. Link numbers to the Campaign

Docs: https://www.plivo.com/docs/messaging/api/10dlc/overview

## Brand

| Method | Path | Purpose |
|---|---|---|
| POST | /10dlc/Brand/ | Create — Tier 2 (one-time fee, non-refundable) |
| GET | /10dlc/Brand/ | List |
| GET | /10dlc/Brand/{brand_id}/ | Get status |

Status flow: `PENDING` → `VERIFIED` (or `FAILED`). Approval by TCR (external), not instant.

Key params: `legal_name`, `brand_type`, `entity_type` (public/private), `tax_id` (EIN), `email`, `phone`, `vertical`, `website`.

Fees (current): ~$4 one-time + $12.50 Authentication+ for public brands (TCR-charged).

## Campaign

| Method | Path | Purpose |
|---|---|---|
| POST | /10dlc/Campaign/ | Register — Tier 2 (monthly + vetting fee) |
| POST | /10dlc/Campaign/Import/ | Import externally-registered (private beta) |
| GET | /10dlc/Campaign/ | List |
| GET | /10dlc/Campaign/{campaign_id}/ | Get status |

Key params: `brand_id`, `usecase` (`LOW_VOLUME`, `MARKETING`, `OTP`, `2FA`, `MIXED`, etc.), `description`, `sample_messages` (array, at least 1).

## Number linking

| Method | Path | Purpose |
|---|---|---|
| POST | /10dlc/Campaign/{campaign_id}/Number/ | Link number(s) — Tier 1 |
| DELETE | /10dlc/Campaign/{campaign_id}/Number/{number}/ | Unlink |

## Gotchas

- Brand vetting fee is one-time but NON-REFUNDABLE — confirm intent
- Campaign approval can take days; SMS to consumers fails until both VERIFIED
- Unregistered US traffic gets blocked/filtered (errors 117, 4720, 4730)
- Number must be active in account before linking
- Don't register at the last minute — bake delays into rollout plans
