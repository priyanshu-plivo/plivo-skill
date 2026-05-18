# Pricing

Get per-country rates for SMS, voice, and phone numbers. Critical for cost estimation before Tier 1/2 actions.

(Note: this endpoint is live but NOT in Plivo's public docs index — discovered via API probing.)

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | /Pricing/?country_iso={ISO} | Get rates for a country — Tier 0 |

`country_iso` is REQUIRED (ISO 3166 alpha-2: `US`, `IN`, `GB`, etc.). Without it returns HTTP 400.

## Response shape

```json
{
  "country": "United States",
  "country_code": 1,
  "country_iso": "US",
  "message": {
    "inbound":  {"rate": "0.00550"},
    "outbound": {"rate": "0.00770"},
    "outbound_networks_list": [{"group_name": "...", "rate": "0.00770"}, ...]
  },
  "voice": {
    "inbound":  {"rate": "..."},
    "outbound": {"rate": "..."},
    "outbound_networks_list": [...]
  },
  "phone_numbers": {
    "local":    {"rate": "..."},
    "tollfree": {"rate": "..."},
    "mobile":   {"rate": "..."}
  }
}
```

Sections may be omitted if Plivo doesn't support that channel in the country.

## Use this for safety/cost gating

Before any Tier 1/2 spend action, fetch pricing and compute estimate:
- SMS: `message.outbound.rate × segments × recipients`
- Voice: `voice.outbound.rate × minutes`
- Number rent: `phone_numbers.{type}.rate` (monthly)

Then show to user per safety.md before proceeding.

## Gotchas

- All rates are STRINGS like `"0.00770"`. Parse before math.
- `outbound_networks_list` can be huge (India has 500+ entries). Use top-level `outbound.rate` unless you need carrier-specific.
- Inbound rate may be `null` if not supported.
- Rates can change — fetch on demand, don't long-cache.
- Country not supported → HTTP 200 with mostly empty payload (not 404). Check for actual rate values.
