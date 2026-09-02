# Input

One pipeline handles all pointer input, regardless of mode. Modes differ only in the final step.

```
pointerdown/move/up (Pointer Events API, setPointerCapture)
  → getCoalescedEvents()        // trackpads report faster than the compositor ticks;
                                // skipping this drops samples and produces polygonal strokes
  → screen→world transform
  → per-mode handler
```

---

## Coalesced events

`getCoalescedEvents()` is not an optimization — it is a correctness requirement.

A trackpad reports position faster than the compositor ticks. A handler that reads only the event it was called with sees one sample per frame and silently discards the rest, and the stroke that results is visibly polygonal. The samples are there; they are simply not in the event you were handed.

---

## Pointer capture

`setPointerCapture` on `pointerdown` keeps the whole gesture bound to the element that received it. A drag that leaves the canvas — off the window edge, over the mode pill, across a text overlay — continues to be delivered, and `pointerup` arrives where it is expected rather than being lost.

---

## Pressure

**The MacBook Force Touch trackpad's pressure is not exposed to Chromium via `PointerEvent.pressure`.** It reports `0` or `0.5` and nothing in between.

Stem therefore uses `perfect-freehand`'s `simulatePressure: true`, which derives width from velocity: fast strokes thin, slow strokes thicken. This reads as natural, and it is the only option available — not a fallback chosen over a better one.

`simulatePressure` is exposed in [Settings](../Features/Settings.md) alongside the other `perfect-freehand` options, but it is required rather than optional. See [Drawing](../Features/Drawing.md) for the full option set.

---

## Trackpad gestures

Two-finger scroll pans. Pinch zooms. `Cmd` + two-finger scroll also zooms.

**The browser must never handle these.** `preventDefault` on `wheel` with `passive: false` — otherwise the page itself scrolls or zooms underneath the canvas, and the two transforms fight.

---

## Keyboard

Keyboard input does not run through this pipeline. It is a separate concern, and the routing rule that matters most is [the text focus rule](../Features/Modes.md#the-text-focus-rule): while a text box has focus, single-letter mode keys are inert.
