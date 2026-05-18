# SIP Trunking (Zentrunk)

Plivo's SIP trunking product for businesses with their own PBX.

Docs: https://www.plivo.com/docs/sip-trunking

## Trunks

| Method | Path | Purpose |
|---|---|---|
| POST | /Zentrunk/Trunk/ | Create trunk — Tier 1 |
| GET | /Zentrunk/Trunk/ | List |
| GET | /Zentrunk/Trunk/{trunk_id}/ | Get |
| POST | /Zentrunk/Trunk/{trunk_id}/ | Update |
| DELETE | /Zentrunk/Trunk/{trunk_id}/ | Delete |

Create params: `name`, `trunk_type` (`inbound` / `outbound` / `both`), `termination_origination` config, IP allowlist, dial prefix rules.

## Calls (CDR)

| Method | Path | Purpose |
|---|---|---|
| GET | /Zentrunk/Call/ | List Zentrunk calls |
| GET | /Zentrunk/Call/{call_id}/ | Get |

Separate from the regular `/Call/` resource — Zentrunk calls have their own CDRs.

## Use case

Enterprise: bring your own PBX (FreePBX, Asterisk, Twilio Flex, etc.), use Plivo for PSTN connectivity. Different pricing model (per-channel or per-minute by region).

Not relevant for most "agent calling someone" use cases — that's regular Voice API.

## Hangup codes

SIP-specific. See https://www.plivo.com/docs/sip-trunking/troubleshooting/zentrunk-hangup-codes for the full list.

## Incoming Carrier (advanced SIP inbound)

| Method | Path | Purpose |
|---|---|---|
| GET | /IncomingCarrier/ | List inbound SIP carriers |
| POST | /IncomingCarrier/ | Create — Tier 1 |
| GET | /IncomingCarrier/{carrier_id}/ | Get |
| POST | /IncomingCarrier/{carrier_id}/ | Update |
| DELETE | /IncomingCarrier/{carrier_id}/ | Delete |

For configuring third-party carriers to deliver inbound SIP calls into your Plivo account. Enterprise-only feature; most users don't need this.
