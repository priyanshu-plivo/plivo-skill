# Plivo XML — Core Verbs

Plivo's voice servers fetch XML from your `answer_url` to control the call. This file covers the 90% case: speak, play, collect input, hangup, redirect.

Docs: https://www.plivo.com/docs/voice/xml/overview

## Root: <Response>

Every doc starts with:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  ...verbs...
</Response>
```
Verbs execute in order. After the last verb (with no Redirect), call hangs up.

## <Speak> — TTS

```xml
<Speak voice="Polly.Joanna" language="en-US">Hello, this is a test.</Speak>
```

Attributes:
- `voice` — `Polly.Joanna`, `Polly.Matthew`, `Polly.Aditi` (en-IN), many AWS Polly voices
- `language` — fallback if voice doesn't specify
- `loop` — repeat N times (0 = infinite, default 1)

Limit ~3000 chars per `<Speak>`. Long content → multiple verbs.

## <Play> — Audio from URL

```xml
<Play loop="1">https://your.cdn/welcome.mp3</Play>
```
Supports: MP3, WAV (16-bit PCM, 8kHz/16kHz mono). HTTPS only.

## <Wait>

```xml
<Wait length="2"/>
```
Pause seconds. Default 1, max 60.

## <GetInput> — DTMF or speech (modern)

```xml
<GetInput action="https://your.app/route" method="POST"
          inputType="dtmf speech" timeout="10" 
          finishOnKey="#" numDigits="4">
  <Speak>Press 1 for sales, 2 for support, or say "agent".</Speak>
</GetInput>
```
Attributes:
- `inputType` — `dtmf`, `speech`, or both (space-separated)
- `action` — URL receiving collected input (Digits or Speech params)
- `method` — GET/POST
- `timeout` — seconds (default 5)
- `finishOnKey` — DTMF that ends collection (default `#`)
- `numDigits` — max digits
- `language` — speech recognition locale (e.g. `en-US`, `hi-IN`)
- `hints` — comma-separated expected phrases

If no input: `action` is still called, with empty Digits/Speech.

## <GetDigits> — DTMF-only (legacy)

```xml
<GetDigits action="/digits" numDigits="1" timeout="5">
  <Speak>Press a digit.</Speak>
</GetDigits>
```
Same as GetInput minus speech. Prefer GetInput for new builds.

## <DTMF> — Send DTMF tones

```xml
<DTMF>1234#</DTMF>
```
Sends DTMF on the call (for downstream IVR navigation).

## <Hangup>

```xml
<Hangup reason="user-requested"/>
```
End the call. Optional `reason` is logged for analytics.

## <Redirect>

```xml
<Redirect method="GET">https://your.app/next.xml</Redirect>
```
Fetch new XML from another URL. Useful for state machines.

## Common pattern: Welcome + menu + route

```xml
<Response>
  <Speak>Welcome to Acme.</Speak>
  <GetInput action="/menu" method="POST" inputType="dtmf" numDigits="1" timeout="5">
    <Speak>Press 1 for billing, 2 for support.</Speak>
  </GetInput>
  <Speak>No input received. Goodbye.</Speak>
  <Hangup/>
</Response>
```

## Gotchas

- XML must start with `<?xml ...?>` declaration (some servers strip it — check)
- `action` URLs are POSTed with form-encoded params, not JSON
- A `<Redirect>` mid-flow stops further verbs in current doc and switches to new URL
- For AI voice agents (no static IVR), use `<Stream>` — see xml/advanced.md
