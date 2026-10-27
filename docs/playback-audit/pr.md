# PR title

fix(video): guard stale recovery and preserve live playback intent

# PR description

A paused or refreshed stream could still be controlled by delayed recovery work.
Resuming an expired paused session deferred itself again; normal resume retained
the paused DVR position. Brief play/buffer bounces also reset the seek budget,
allowing repeated recovery seeks.

Guard delayed work by session/controller/intent, serialize seek-then-play, retain
recovery budgets through short bounces, and refresh expired sessions on explicit
resume. Patch native iOS stall/seek/interruption callbacks and forward-only live
seek targets. Dispose native shared players by controller ID after their final
platform view disappears (relevant to PiP sleep-timer audio). Preserve Android
pause intent on expired live windows. Correct the Android HLS rewriter's target
duration for mixed-length segments and preserve malformed/non-Twitch input.

Also confirm offline state only from explicit empty stream responses, and batch
pinned-channel requests at Helix's 100-ID limit. Refs #597, #599, #439; partially
addresses #573. These references are not claims that all native symptoms are fixed.

The resolved native player is vendored at its existing 19c6275 revision in a
separate baseline commit, with origin/license and preserved iOS symlinks. This
makes the actual dependency fixes reproducible without write access to its fork.
Other dependency resolutions, buffer settings and playback rates are unchanged.

Validation: 389 root Flutter tests pass; analysis clean; code generation succeeds
with no generated-file diff; touched Dart formatting passes. New regression cases
fail against the corresponding baseline. The production Kotlin playlist harness
fails before and passes after the fix. Complete issue/PR inventory, dispositions,
logs, acceptance criteria and manual steps are included in docs/playback-audit.

Native builds and device playback remain unverified: Linux has no Android SDK or
Xcode/mobile devices. No measured latency or A/V-drift improvement is claimed.
Please run the documented Android/iOS matrix before release, especially PiP sleep
timer, interruption permissions, broadcaster restart and rendition transitions.
