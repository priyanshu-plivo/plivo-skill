# Plivo XML — Advanced Verbs

Routing, bridging, recording, and AI voice agents.

## <Dial> — Bridge A-leg to B-leg

```xml
<Dial action="/dial-result" method="POST" 
      callerId="+1XXX" timeLimit="3600" timeout="30"
      hangupOnStar="true" record="false">
  <Number>+1YYY</Number>
</Dial>
```

Attributes:
- `action` — POSTed after dial completes (with `DialStatus`, `DialBLegHangupCause`, etc.)
- `callerId` — your Plivo number or verified caller ID
- `timeLimit` — max bridge duration seconds
- `timeout` — ring timeout
- `hangupOnStar` — caller pressing `*` ends B-leg
- `confirmSound` — URL of audio played to B-leg when answered (e.g. "Press 1 to accept")
- `confirmKey` — DTMF to confirm acceptance
- `record` — auto-record
- `redirect` — true bridges A&B; false keeps A on `<Dial>` waiting

Children (one or more):
- `<Number>` — PSTN destination
- `<User>` — SIP endpoint

## <Number> (child of Dial)

```xml
<Number sendDigits="wwww1234" gateways="dispatcher,carrier1">+1YYY</Number>
```
Attributes:
- `sendDigits` — DTMF after answer (use `w` for ~500ms pause)
- `sendOnPreanswer` — send DTMF before answer
- `gateways` — comma-separated routing hints
- `gatewayCodecs`, `gatewayTimeouts`, `gatewayRetries`

## <User> (child of Dial)

```xml
<User>sip:bob@phone.plivo.com</User>
<User>sip:agent@your-pbx.example.com</User>
```
Bridge to a SIP endpoint (Plivo SIP user or external SIP).

## <Conference>

```xml
<Conference action="/conf-result" maxMembers="10" record="true"
            startConferenceOnEnter="true" endConferenceOnExit="false"
            waitSound="https://your.cdn/hold.mp3"
            enterSound="beep:1" exitSound="beep:1">
  conference-room-name
</Conference>
```

Attributes:
- `maxMembers` — cap participants
- `record` — auto-record
- `startConferenceOnEnter` — false = mute-and-wait until host joins
- `endConferenceOnExit` — true = entire room ends when this caller leaves (host pattern)
- `waitSound` — music on hold (URL or `beep:N`)
- `enterSound` / `exitSound` — chime on join/leave (`beep:1`, `beep:2`, or URL)
- `muted` — start muted
- `floorEvent` — POST when active speaker changes
- `digitsMatch` — comma-separated DTMF patterns to trigger callbacks
- `callbackUrl` — webhook for DTMF events
- `relayDTMF` — relay DTMF to other participants

## <Record>

```xml
<Record action="/recorded" method="POST" maxLength="60"
        finishOnKey="#1234567890*"
        playBeep="true" recordSession="false"
        transcriptionType="auto" transcriptionUrl="/transcribed"/>
```

Attributes:
- `maxLength` — seconds (default 60, max 14400)
- `finishOnKey` — any DTMF in this string ends recording
- `playBeep` — beep before recording starts
- `recordSession` — true to record entire call from this point (not just one segment)
- `transcriptionType` — `auto` (Plivo) or `hybrid`
- `transcriptionUrl` — webhook for transcript
- `redirect` — true to continue XML after, false to hangup after

`action` POSTed with `RecordUrl`, `RecordingDuration`, `Digits`.

## <Stream> — WebSocket audio (AI voice agents)

```xml
<Stream bidirectional="true" keepCallAlive="true"
        contentType="audio/x-mulaw;rate=8000"
        audioTrack="inbound"
        statusCallbackUrl="/stream-status" statusCallbackMethod="POST">
  wss://your.agent/ws
</Stream>
```

Attributes:
- `bidirectional` — true to allow sending audio back to caller
- `keepCallAlive` — keep stream open past end of this verb
- `contentType` — `audio/x-mulaw;rate=8000` (default) or `audio/x-l16;rate=16000`
- `audioTrack` — `inbound`, `outbound`, `both`
- `extraHeaders` — additional headers on WebSocket upgrade
- `statusCallbackUrl`, `statusCallbackMethod` — lifecycle webhook

Primary path for AI voice agents. See api/audio-streams.md for the REST equivalent (POST on active call). See webhooks/stream.md for the lifecycle events and message format.

## <Message> — Send SMS from voice flow

```xml
<Message src="+1XXX" dst="+1YYY">Thanks for calling. Reference: ABC123.</Message>
```
Send SMS during/after a voice call. Same per-segment pricing as POST /Message/.

## <PreAnswer>

```xml
<PreAnswer>
  <Speak>Please wait, connecting you now.</Speak>
</PreAnswer>
<Dial>
  <Number>+1YYY</Number>
</Dial>
```
Wraps verbs that execute BEFORE the call is technically answered (early media). Useful for "ringback" UX or pre-answer messages.

## <SIPDigits>

```xml
<SIPDigits>1234</SIPDigits>
```
Send DTMF via SIP INFO method (some carriers prefer this over inband).

## Gotchas

- `<Stream>` must come AFTER a verb that answers the call (e.g., not before `<Dial>`); use `<PreAnswer>` if pre-answer needed
- `<Dial>` `record="true"` records the bridged call but stops if A-leg hangs up
- `<Conference>` participants joining identically-named room enter the same conference
- DTMF over RTP (`<DTMF>`) vs SIP INFO (`<SIPDigits>`) — carrier-dependent which works
