Status: Open

# Stack

What Stem is built with, now that it runs on desktop and mobile with full parity.

## Settled

- Flutter is the working choice, held until the spike passes.
- v1 targets Android, macOS, and Windows.
- The reason is one renderer everywhere. Flutter draws every pixel itself rather than using each platform's web engine, so text and drawing behave identically across platforms — the property Electron was originally chosen for, which no web stack can keep once a phone is in scope.
- The GitHub integration uses GitHub's API. No git runs on the device.
- Pressure is simulated on every device. Stylus pressure and tilt are not used.

## The spike

Three tests, each on all three target platforms. Built to answer, then discarded.

1. **Ink.** Strokes drawn with `perfect_freehand` (official Dart port) feel immediate and smooth: finger on the Android phone, trackpad on macOS, mouse or trackpad on Windows.
2. **Text.** A rich text box with Stem's supported marks edits correctly, keeps its marks, and zooms with the canvas. Candidates: `appflowy_editor`, `super_editor`, `flutter_quill`.
3. **Clipboard.** A PNG image reaches the system clipboard and pastes into another app. Candidate: `super_clipboard`.

All three pass: Flutter is confirmed and Architecture is rewritten. Text fails: the fallback is a web app in Tauri 2, accepting a different web engine per platform.

## Open questions

- What replaces the reactivity rule: how scene state stays out of the framework's rebuild cycle.
- How the three-layer renderer maps onto Flutter's painting and repaint boundaries.
- SVG export without a browser document to serialise.
- Secure storage for the GitHub token on each platform.

## Findings

- **Web stacks cannot keep one engine on mobile.** Every mobile shell uses the system web engine — WebKit on iOS, the system webview on Android.
- **tldraw would remove the canvas engine risk**, but production use needs a paid licence or a visible watermark, and its default interaction model is the one Stem's principles reject.

## Dependencies

- **Requires:** none.
- **Required by:** Touch Input — the input events available. Sync — the local store.
