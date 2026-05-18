# Workflow: Send WhatsApp Template Message

## Pre-reqs

- WABA onboarded via Plivo Console (manual — see not-supported.md)
- At least one Meta-approved template (created in WhatsApp Manager, NOT via Plivo)
- "Sync Templates from WhatsApp" run in Plivo Console after Meta approval

## Step 1 — Find your template

In Plivo Console → WhatsApp → Templates. Note:
- `name` (case-sensitive)
- `language` code (e.g. `en_US`)
- Parameter count per component

## Step 2 — Send

```bash
curl -X POST "https://api.plivo.com/v1/Account/$PLIVO_AUTH_ID/Message/" \
  -u "$PLIVO_AUTH_ID:$PLIVO_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "src":"+<your_waba_number>",
    "dst":"+<recipient>",
    "type":"whatsapp",
    "template":{
      "name":"order_confirmation",
      "language":"en_US",
      "components":[
        {
          "type":"body",
          "parameters":[
            {"type":"text","text":"John"},
            {"type":"text","text":"#1234"}
          ]
        }
      ]
    }
  }'
```

## Parameter count

Must match the template EXACTLY. Off-by-one = error `132000 invalid template parameters`.

## Session messages (free-form)

After the user replies to you, you have a 24-hour window for free-form text without a template:

```json
{
  "src":"+<waba>",
  "dst":"+<recipient>",
  "type":"whatsapp",
  "text":"Reply within the 24h window"
}
```

## Pricing

Per-conversation (24h window), not per-message. Meta categorizes:
- Utility (txn updates): cheapest
- Authentication (OTP): cheap
- Marketing: most expensive
- Service (user-initiated reply): free

## Common errors

- `132000` — template not found, OR parameter mismatch
- `131047` — outside 24h window, you must use a template
- `131056` — `dst` is not a valid WhatsApp user
- "template not synced" — run Sync Templates in Console

## Cannot do via API

- Create / submit / delete templates (Meta's WhatsApp Manager only)
- Onboard a new WABA / phone number (Plivo Console Embedded Signup)

See not-supported.md.
