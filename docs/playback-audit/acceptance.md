# Playback acceptance and measurement plan

Baseline: Frosty main 3ea6ccd43508fd162b045c7b0a4760716d6bcfd1;
player 19c627547c227f1fc9895a5a62839e64fec689b3. Defined before behavior changes.

## Automated acceptance

- A pause, chat-only toggle, disposal or newer load invalidates pending recovery.
- Seek completes before play; concurrent triggers cannot issue overlapping recovery seeks.
- Failed seek does not start playback; failures are observable and retries remain capped.
- Resuming an expired paused session obtains a fresh session once.
- Invalid or stale latency results cannot update state or trigger recovery.
- API failures cannot establish that a stream is offline.
- Live-edge seeks do not rewind a playhead already nearer the edge within the current window.
- HLS prefetch promotion preserves sequence/discontinuity/ad tags; target duration
  accommodates every advertised segment, including mixed-duration fixtures.

## Device acceptance (not established by unit tests)

Use the same phone/OS, stream/quality, network shaping, account, build mode and
30-minute interval for baseline and candidate. Run three repeats. Record model,
OS, build SHA, player SHA, network profile and raw observations. Use a broadcaster
feed with a visible UTC clock and repeated simultaneous flash/beep markers;
record the screen and system audio with a shared capture clock.

| Metric | Definition | Acceptance goal |
|---|---|---|
| Startup (seconds) | Tap stream to first rendered moving frame plus audible sound | No regression >1s in median across 3 runs |
| Delay behind live (seconds) | Visible source clock vs independently synchronized reference clock | Report median/p95; improvement requires measured reduction without stalls |
| Recovery (seconds) | Network restored/interruption ended to moving video and synchronized audio | <=10s for brief loss; otherwise bounded retry or visible error, no endless churn |
| A/V drift (milliseconds) | Flash presentation time minus beep onset in same recording | Absolute drift <=100ms; no independent track jumps |
| Frame stalls | Frozen picture >1s while audio/clock advances | Zero in sustained stable-network run |
| User intent | Pause during each delayed operation | No automatic restart, including remote controls and PiP |

Run iOS and Android: sustained playback; 3 loss/reconnect cycles (5s/15s/30s);
constrained bandwidth and jitter; Auto and fixed quality changes; stream switches;
pause 30s/120s then resume; background/foreground; audio interruptions (call/Siri/AirPods);
PiP sleep timer; broadcaster restart; rendition transitions. Capture method and
full matrix in the original acceptance document.
