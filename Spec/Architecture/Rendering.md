# Rendering

Three stacked canvases, identically sized and absolutely positioned. Each has its own redraw trigger, and the split between them is the single most important performance decision in Stem.

---

## The three layers

| Layer | Redraw trigger | Contents |
|---|---|---|
| **Background** | viewport change only | dot grid |
| **Scene** | scene mutation or viewport change | all committed objects |
| **Overlay** | every frame during interaction | wet ink, marquee, hint labels, selection outline |

**Wet ink lives on the overlay.** While a stroke is being drawn, only the overlay redraws — the scene canvas is untouched. The cost of drawing a stroke is therefore independent of how much is already on the board, which is what keeps a dense board as responsive as an empty one.

The same holds for every other transient mark: a marquee drag, hint letters, and the selection outline all cost one overlay redraw and never touch committed content.

---

## Context and display

- Get contexts with `{ desynchronized: true, alpha: true }`.
- Handle `devicePixelRatio` explicitly: set the canvas backing store to `cssSize * dpr` and scale the context. Getting this wrong produces soft strokes on every retina display, which is every display Stem will run on.

---

## The loop

A single `requestAnimationFrame` loop with a dirty flag per layer. **If nothing is dirty, do nothing** — no clear, no draw, no work at all.

An idle board must cost nothing. This is why `backgroundThrottling: false` is safe to set in [the Electron config](Architecture.md#electron-config-day-one): the loop is already doing nothing when there is nothing to do, so disabling the platform's throttle does not leave a busy loop running behind an unfocused window.

---

## Stroke rendering

Strokes store a simplified point array, not an outline — see [Data Model](../Data%20Model.md#stroke). The `perfect-freehand` outline is regenerated at render time.

This is what allows stroke width to respond to zoom and to weight changes, and it is why ink scales with zoom rather than staying a constant screen width. A mark on the canvas behaves like a mark, not like a UI element.

---

## Deferred: tile caching

**Condition:** full scene redraw drops below 60fps.

**Change:** bucket committed objects into 512×512 world-space tiles rendered to offscreen canvases, and blit only the visible tiles.

Not built until the condition is observed. A tile cache adds an invalidation problem to every mutation path, which is a real cost paid against a hypothetical one.
