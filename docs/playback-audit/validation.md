# Validation results and limits

| Check | Result | Notes |
|---|---|---|
| 389 root Flutter tests | Pass | All pass after changes |
| Analysis | Clean | No issues |
| Code generation | Pass | Generated outputs regenerated; no committed `.g.dart` differences. Final repeat succeeds |
| Formatting | Pass | 9 touched Dart files checked with `dart format --output=none --set-exit-if-changed`, 0 changes |
| Production Kotlin playlist harness | Fail before / pass after | Compiles actual production rewriter, synthetic mixed-duration/discontinuity/ad fixture, prefetch ordering, pass-through and malformed durations |
| Android release APK | **Blocked** | Flutter downloaded engine artifacts, then reported “No Android SDK found”; no native compilation occurred |
| Android observer/JUnit tests | **Not run** | Android SDK/Gradle platform environment unavailable; added pause regression awaits platform CI |
| Swift grammar syntax | Pass (limited) | Four changed Swift files parsed with tree-sitter 0.25.2 / Swift grammar 0.7.4; no syntax errors. This is not compilation or typechecking |
| iOS release / Swift typecheck | **Unavailable** | Linux has no Xcode/Apple SDK; Flutter rejects iOS invocation before compilation |
| Device playback / A/V / live latency | **Not measured** | No mobile device/emulator or configured Twitch test session |
| Final diff | Reviewed | Fix diff passes whitespace check with `core.whitespace=cr-at-eol` (existing Dart CRLF). The exact vendor import retains upstream trailing whitespace; full base diff therefore reports inherited whitespace |

Kotlin harness used JDK 17 and checksum-verified Kotlin compiler 2.4.0. The app's
Android build remains on its existing toolchain; this standalone JVM check does
not establish Media3 parsing, native integration or decoder behavior. Reproduce:

```bash
java -cp "$KOTLIN_COMPILER_HOME/lib/*" org.jetbrains.kotlin.cli.jvm.K2JVMCompiler \
  -kotlin-home "$KOTLIN_COMPILER_HOME" \
  third_party/better_native_video_player/android/src/main/kotlin/com/huddlecommunity/better_native_video_player/hls/TwitchPrefetchPlaylist.kt \
  tool/PlayerPlaylistRegression.kt -include-runtime -d /tmp/playlist-regression.jar
java -jar /tmp/playlist-regression.jar
python3 tool/verify_history_audit.py /path/to/extracted/audit
```

The delivered archive includes before/after logs and the full audit source text.
Passing controller tests confirms method-call ordering and ownership, not native
resource release or rendered video. No startup, broadcast delay, recovery-time
or A/V-drift numbers were inferred from these tests. All four device metrics
remain unmeasured; acceptance definitions and capture method are in acceptance.md.
