# Ideas

A running list of ideas worth revisiting. Nothing here is committed to the spec — these are possibilities, not decisions.

An idea that outgrows this file is promoted to `Research/`, where it is developed in parallel to the spec. Promoted ideas are removed from here — their frozen originals live beside the document they seeded, as `Origin.md`. See [CLAUDE.md](CLAUDE.md) for the research conventions; the directory appears when the first topic is written.

The [v1 non-goals](Spec/Design%20Principles.md#v1-non-goals) are a different thing: those are exclusions that make v1 finishable, and most of them should stay excluded. An entry appears here only when there is a reason to think it might not.

---

## Frames

Deferred to v2 and already carrying a slot in the [insert menu](Spec/Features/Insert%20Menu.md).

A frame is a bounded region on the canvas — a way to say "this cluster is a thing" without grouping, which v1 refuses. It is the closest Stem gets to structure without acquiring layers, and it is the natural unit for export: a frame is a selection you drew once and can keep exporting.

Worth revisiting once export has been used on real work for a while. If the answer to "which cluster do I copy out" turns out to be stable across sessions, a frame is the thing that remembers it.

---

## Segment-level erase

The [erase granularity](TASKS.md) question is currently a v1 decision between deleting a whole stroke on contact and splitting it at the erased segment, and whole-stroke is the simpler answer.

Segment-splitting is the one worth parking rather than settling. It matches how a real eraser behaves, and the reason to defer it is that a split turns one stroke into two — each needing its own bounds, its own `originalLength`, and its own binding resolution. That is a data-model consequence, not an interaction preference.

Revisit if whole-stroke erase turns out to be the thing that makes people redraw rather than correct.

---

## Board content search

The [board grid](Spec/Features/Board%20Grid.md) searches by title only. Boards are named by hand, which means they are named badly, which means title search finds the board you remember naming and not the one you are looking for.

Text boxes hold searchable content and are the obvious index. Shape labels are another. The question is whether that is enough to be useful, given that the most memorable thing on a board is usually the drawing.

Worth revisiting once there are enough boards that the grid stops being a complete list you can scan.

---

## Tauri

The [Electron decision](Spec/Architecture/Architecture.md#why-electron-over-tauri) is deliberate and the reasoning holds: two webview engines mean two sets of text metrics and clipboard behavior, on an app whose text is HTML and whose export is three clipboard formats.

It is parked rather than closed because the `PlatformAdapter` exists specifically to keep it cheap, and because the cost being paid — roughly 110MB of install and 120MB of idle RAM — is the kind that stops being acceptable at some point without any single moment where it changes.

Revisit if the install size becomes a reason not to use the app, or if WebView2 and WKWebView converge enough that the divergence argument stops being true.
