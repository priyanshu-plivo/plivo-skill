# Voice Call Webhooks

## Answer URL (XML response, not webhook)

Plivo fetches your `answer_url` when the call connects. Your response MUST be Plivo XML (see xml/core.md). Not a webhook in the async-response sense.

Params Plivo sends:
- `CallUUID` — Plivo's UUID
- `From`, `To`
- `Direction` — `outbound` (you placed) / `inbound` (someone called you)
- `CallStatus` — `in-progress` / `ringing`
- `Event` — `Answer`

## Hangup URL

POSTed when the call ends.

Form-encoded:
- `CallUUID`
- `From`, `To`
- `Direction`
- `AnswerTime` — ISO 8601 (null if not answered)
- `EndTime` — ISO 8601
- `Duration` — total seconds (rounded)
- `BillDuration` — billed seconds
- `BillRate` — STRING (per-minute rate)
- `TotalCost` — STRING
- `HangupCause` — see list below
- `HangupSource` — `Caller` / `Callee` / `Plivo`
- `Event` — `Hangup`
- `STIRVerification` — `Verified` / `Not Verified` / `Not Applicable` (US)
- `StirAttestation` — `A` / `B` / `C` (US STIR/SHAKEN)

## Ring URL

POSTed when B-leg ringing starts (outbound).

Params:
- `CallUUID`, `From`, `To`
- `CallStatus` — `ringing`
- `Event` — `Ring`

## Machine Detection URL

POSTed when AMD finishes (only if `machine_detection: true` on /Call/ POST).

Params:
- `CallUUID`
- `MachineDetectionResult` — `human` / `machine` / `unknown` / `not-applicable`
- `MachineDetectionDuration` — ms

Return XML to continue the flow (or empty for default).

## Fallback URL

Used when `answer_url` errors or returns invalid XML. Same params as answer_url. Must return valid XML.

## HangupCause values

| Cause | Meaning |
|---|---|
| `NORMAL_CLEARING` | Clean end (most common) |
| `NO_ANSWER` | ring_timeout exceeded |
| `USER_BUSY` / `BUSY` | Callee busy signal |
| `CALL_REJECTED` | Callee actively rejected |
| `FAILED` | Generic failure |
| `INVALID_NUMBER_FORMAT` | Bad `to` parameter |
| `CONGESTION` | Carrier capacity / routing |
| `RECOVERY_ON_TIMER_EXPIRE` | Call dropped mid-conversation |
| `MEDIA_TIMEOUT` | RTP stopped flowing |
| `ORIGINATOR_CANCEL` | Caller hung up before answer |
| `UNALLOCATED_NUMBER` | Number doesn't exist |
| `NUMBER_CHANGED` | Number was reassigned |
| `UNREACHABLE` | Network unreachable |

## Gotchas

- Hangup webhook fires even for unanswered calls (Duration=0, BillDuration=0)
- `AnswerTime` is null for unanswered — check before parsing
- All money/duration fields are STRINGS — parse before math
- Retries are limited for voice (vs more for SMS) — your endpoint must be fast
