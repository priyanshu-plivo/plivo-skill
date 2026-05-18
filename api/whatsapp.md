# WhatsApp

WhatsApp sending via Plivo. Plivo wraps Meta's WhatsApp Cloud API.

Docs: https://www.plivo.com/docs/messaging/concepts/whatsapp/

## Endpoints

WhatsApp uses the regular `Message` API with `type: whatsapp`.

| Method | Path | Purpose |
|---|---|---|
| POST | /Message/ | Send WhatsApp message |
| GET | /Message/?message_type=whatsapp | List WhatsApp messages |

There is NO endpoint to create/submit templates — that's done in Meta's WhatsApp Manager. See `not-supported.md`.

## Send template message

Templates must be pre-approved by Meta. Sync them in Plivo Console first ("Sync Templates from WhatsApp").

```json
{
  "src": "+WABA_NUMBER",
  "dst": "+1234567890",
  "type": "whatsapp",
  "template": {
    "name": "approved_template_name",
    "language": "en_US",
    "components": [
      {"type": "header", "parameters": [{"type":"media","media":"https://..."}]},
      {"type": "body", "parameters": [
        {"type":"text","text":"John"},
        {"type":"text","text":"Order #1234"}
      ]},
      {"type": "button", "sub_type":"url", "index":"0",
        "parameters":[{"type":"text","text":"track-1234"}]}
    ]
  }
}
```

## Send session message (free-form)

Allowed ONLY within 24h after the user replied to you (the "customer service window").

```json
{
  "src":"+WABA","dst":"+1YYY","type":"whatsapp",
  "text":"Free-form reply within the 24h window"
}
```

## Send media

For images, docs, videos, audio:
```json
{
  "src":"+WABA","dst":"+1YYY","type":"whatsapp",
  "media_url":"https://your.cdn/image.jpg",
  "caption":"optional caption"
}
```

## Gotchas

- Template name must match Meta-approved name EXACTLY (case-sensitive)
- Parameter count must match exactly — extras/missing fail with `132000`
- 24h customer service window: outside it, only templates allowed (errors with `131047`)
- Pricing is per-conversation (24h window), not per-message, categorized by Meta:
  - Utility (transactional updates): cheapest
  - Authentication (OTP): cheap
  - Marketing: most expensive
- Plivo can't auto-discover new Meta templates — manual "Sync Templates" in Console
- Phone number must be onboarded to WABA via Console (manual; see not-supported.md)
