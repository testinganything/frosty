# Frosty playback investigation

Audit snapshot: main `3ea6ccd43508fd162b045c7b0a4760716d6bcfd1`, audit started
2026-10-27T00:20:34.860Z. Retrieval and discussion reading completed before any
application behavior changes. The restored checkout matches this SHA.

## Coverage

| Inventory | Count | Discussion coverage |
|---|---:|---|
| Issues | 389 (170 open, 219 closed) | Full descriptions and 365 conversation comments |
| Pull requests | 210 (151 merged, 26 closed unmerged, 33 open) | Full descriptions, 90 conversation comments, 27 review submissions, 13 threads / 16 inline comments including replies, resolved and outdated threads |

The open PR count includes 2 drafts; the closed unmerged count includes 1 draft.
Enumeration followed all REST pages, including explicit empty terminal pages.
All 455 conversation comments match the inventory's comment totals. Every PR's
reviews and threads were retrieved. The complete REST inline-comment ID set
matches the GraphQL thread-reply set. The original API responses, reading
transcripts, provenance/deduplication records and retrieval manifests are supplied
in the audit archive. Repetitive identical dependency-bot paragraphs were read
once with per-item provenance; original item bodies are retained.

All 158 GitHub-hosted attachments were retrieved after retries. Images were
inspected via contact sheets (reporter statistics also full size); videos were
reviewed as eight visual samples each. Audio tracks and full temporal playback
were not reviewed. These samples are evidence of the reported UI, not A/V timing
measurements. Large attachment binaries are excluded from the compact archive;
their exact source URLs, retrieval results, samples and manifests are retained.

Explicit gaps: #53 returns 410 (deleted); #277, #348, #399, #470, #471 and #569
return 404 and are absent from enumeration. Their contents were not read.
External reproduction gaps: #264 Pastebin log returns 403 and the linked OnePlus
page supplies only a script shell; #279 Dropbox folder inaccessible; #139/#317
Imgur resources return 429; #488 Imgur album inaccessible; #331 Samsung shared
link expired; #492 YouTube page retrieved but the video was not watched. No
implementation depends on claiming those materials were read. Truncated
Dependabot boilerplate authored in PR bodies is retained as authored.

Separate final ledgers: [issues](issues-ledger.md), [PRs](prs-ledger.md), and
[machine-readable ledger](ledger.json). Every item has an explicit disposition.

## Findings (code races confirmed)

Delayed recovery work could still drive a paused or refreshed stream. Resuming
an expired paused session re-deferred itself; normal resume retained the paused
DVR position. Brief play/buffer bounces reset the seek budget, permitting
repeated recovery seeks. Native iOS stall/seek/interruption callbacks and
forward-only live seek targets needed patching. Shared players required disposal
by controller ID after final platform-view teardown (PiP sleep-timer audio).
Android pause intent on expired live windows and the HLS rewriter's target
duration for mixed-length segments needed correction; malformed/non-Twitch input
must be preserved. Offline state must be confirmed only from explicit empty
stream responses; pinned-channel requests must batch at Helix's 100-ID limit.

These races can plausibly produce unwanted seeks/restarts. They do not prove the
root cause of every reported symptom (frozen video with continuing audio,
independent track jumps, long-term A/V drift, decoder stalls, live latency).

## What was established

- 389 root Flutter tests pass after the changes.
- Analysis is clean; code generation succeeds with no generated-file diff.
- Touched Dart formatting passes.
- New regression cases fail against the corresponding baseline.
- Production Kotlin playlist harness fails before and passes after the fix.
- The resolved native player is vendored at its existing 19c6275 revision
  (separate baseline commit) with origin/license and preserved iOS symlinks.
  Path dependency + lockfile select that source reproducibly; other resolved
  packages are unchanged.

## What is not established

There is no device, emulator, Xcode, Android SDK or Twitch test account here.
The user's frozen-video/audio-continuing, independent track jumps, long-term A/V
drift, decoder stalls and actual live latency were not reproduced in native
playback. The code races above are confirmed separately and can plausibly cause
unwanted seeks/restarts; they do not prove the root cause of every symptom.

Outstanding hypotheses include rendition timestamp/discontinuity mismatches,
decoder/surface stalls despite position updates, network/CDN behavior, and
program-date-time/device-clock errors. In particular, the linked player PR #2
reports a misleading ~29s latency after broadcaster restart despite a much
smaller observed delay. The existing bounded high-latency policy still needs a
measurement source that distinguishes broadcast-clock latency from seekable-edge
distance. The rewriter's target duration can still change across playlist
updates as segment maxima change; validate this with actual Media3 timelines.
No measured playback improvement is claimed.

Other evaluated work remains explicit in the ledgers: #594 needs user-emote scope
migration/account testing; #604's unconditional moderator scopes conflict with
intentional opt-in; #577's background service is a feature needing lifecycle and
platform validation; #595 needs native iPad gesture reproduction; #600 needs a
physical keyboard/framework check. Do not merge these proposals blindly.

See [acceptance and device steps](acceptance.md) and [validation](validation.md).
Interruption handling follows Apple's [audio-session guidance](https://developer.apple.com/library/archive/documentation/Audio/Conceptual/AudioSessionProgrammingGuide/HandlingAudioInterruptions/HandlingAudioInterruptions.html):
without the system's resume hint, playback waits for user action. This intentional
behavior change still needs call/Siri/AirPods testing.

Primary references: [AVPlayer seek completion](https://developer.apple.com/documentation/avfoundation/avplayer/seek(to:completionhandler:)-75bls),
[RFC 8216 target duration](https://www.rfc-editor.org/rfc/rfc8216.html#section-4.3.3.1),
[Frosty #597](https://github.com/tommyxchow/frosty/issues/597),
[Frosty #599](https://github.com/tommyxchow/frosty/issues/599),
[player PR #2](https://github.com/tommyxchow/flutter-native-video-player/pull/2).
