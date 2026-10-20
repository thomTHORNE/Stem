# Ideas

A running list of ideas worth revisiting. Nothing here is committed to the spec — these are possibilities, not decisions.

An idea that outgrows this file is promoted to `Research/`, where it is developed in parallel to the spec. Promoted ideas are removed from here — their frozen originals live beside the document they seeded, as `Origin.md`. See [CLAUDE.md](CLAUDE.md) for the research conventions.

The [v1 exclusions](Spec/Scope.md#excluded) are a different thing: those are exclusions that make v1 finishable, and most of them should stay excluded. An entry appears here only when there is a reason to think it might not.

---

## Frames

Deferred to v2 and already carrying a slot in the [insert menu](Spec/Features/Insert%20Menu.md).

A frame is a bounded region on the canvas — a way to say "this cluster is a thing" without grouping. It is the closest Stem gets to structure without acquiring layers, and it is the natural unit for export: a frame is a selection you drew once and can keep exporting.

Worth revisiting once export has been used on real work for a while. If the answer to "which cluster do I copy out" turns out to be stable across sessions, a frame is the thing that remembers it.

---

## Segment-level erase

The [erase granularity](TASKS.md) question is currently a v1 decision between deleting a whole stroke on contact and splitting it at the erased segment, and whole-stroke is the simpler answer.

Segment-splitting is the one worth parking rather than settling. It matches how a real eraser behaves, and the reason to defer it is that a split turns one stroke into two — each needing its own bounds, its own `originalLength`, and its own binding resolution. That is a data-model consequence, not an interaction preference.

Revisit if whole-stroke erase turns out to be the thing that makes people redraw rather than correct.

---

## Tauri

The [Electron decision](Spec/Architecture/Architecture.md#why-electron-over-tauri) is deliberate and the reasoning holds: two webview engines mean two sets of text metrics and clipboard behavior, on an app whose text is HTML and whose export is three clipboard formats.

It is parked rather than closed because the `PlatformAdapter` exists specifically to keep it cheap, and because the cost being paid — roughly 110MB of install and 120MB of idle RAM — is the kind that stops being acceptable at some point without any single moment where it changes.

Revisit if the install size becomes a reason not to use the app, or if WebView2 and WKWebView converge enough that the divergence argument stops being true.

---

## Grouping

Binding several objects into a unit that selects and moves together and is remembered in the file. Multi-select today is transient: it is forgotten as soon as the selection changes.

Cognition blocks are where things that belong together now go, and Frames is the parked answer for a region of freeform content. Grouping would be a third mechanism for the same need, with costs of its own: a tree in the data model, and an extra step to edit one object inside a group.

Worth revisiting if freeform clusters keep being re-selected by hand to move them together, and neither a block nor a frame fits.

---

## Board templates

A template would start a board with structure already on it, rather than empty.

Cognition types cover the common case: a recurring way of working gets a structure, placed as a block. A board template would be a second, coarser answer to the same need, and the first is not built yet.

Worth revisiting if the same arrangement of blocks keeps being set up by hand on new boards.

---

## Presentation mode

Stepping through a board as a sequence of views, to show it to someone.

Stem is a personal tool for thinking. Showing thinking to someone else is a different job from doing it, and a board built for the second is arranged differently from one built for the first.

Worth revisiting if boards turn out to be shown to others often enough that exporting them is the friction.
