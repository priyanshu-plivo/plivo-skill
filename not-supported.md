# Plivo Operations NOT Available via API

These have no Plivo REST endpoint. Route the user to the Plivo Console / support, or stop the task and explain.

## Account & Billing

- Account recharge / add funds (wire transfer, ~3 business days)
- Payment method management (cards, billing details)
- Plan upgrade / downgrade (sales-led)
- Refund requests (support ticket)
- API key / Auth Token rotation
- 2FA / login security
- Team / user / role management
- Audit logs
- Funds transfer between main ↔ subaccount (subaccount CRUD exists; money movement does NOT)
- Credit limits on subaccounts

## Compliance & Regulatory

- Number porting (manual console flow, 5–20 business days, LOA upload)
- KYC / STIR-SHAKEN identity submission
- Verified Caller ID (manual SMS/voice verification — POST creates request but verification is interactive)
- CNAM registration
- Sender ID registration (non-US, country-specific manual)
- 10DLC TCR final approval (Plivo API submits; TCR approves externally)
- Toll-free verification approval (submission API exists; approval external by carrier)

## WhatsApp (Meta-gated)

- WABA onboarding / Meta business verification (Plivo Console Embedded Signup flow)
- WhatsApp template creation (must be done in WhatsApp Manager, NOT via Plivo)
- Template approval (Meta, up to 24h)
- Display name changes
- Plivo only SYNCS approved templates from Meta (read-only after Meta approval)

## Voice geo-permissions

- Enabling/disabling international destinations (Voice → Geo-Permissions) is Console-only. Without enabling a destination country, calls return HTTP 403 `"Calls to this destination region are barred."`

## Support & Observability

- Plivo support ticket creation (no API)
- Real-time call interception / supervisor whisper (partial via PHLO XML, no direct REST control)
- Webhook signing key rotation
