# Shapes

## What it is

Two ways to get a clean geometric primitive from a hand-drawn stroke. **Neither involves waiting.**

Recognition never fires on a timer, on release, or on a confidence threshold. It fires when the user asks for it — which is what makes "messy is a valid state" survivable in practice. A wobbly box stays a wobbly box until you decide otherwise.

See [Data Model — Shape](../Data%20Model.md#shape) for the field definitions.

---

## Behavior

### `Shift` while drawing — deliberate

Hold `Shift` before or during the stroke. On release the stroke is discarded and replaced by its best-fit primitive.

Fully deterministic, zero false positives, zero latency. Use it when you already know you want a clean box.

### `Tab` after drawing — retroactive

Press `Tab` to convert the selected object — or, if nothing is selected, the most recently created one — to its best-fit primitive.

Press `Tab` again to cycle through the next-best candidates:

```
rect → roundRect → ellipse → diamond → triangle → line → arrow → original
```

This is the core of the **draw fast, tidy later** workflow. Because it is an ordinary command, `Cmd+Z` restores the original stroke — see [Command Layer](../Architecture/Command%20Layer.md).

`original` closing the cycle means the tidy gesture is never a one-way door: cycling all the way round returns the stroke you drew.

### Recognition algorithm

Runs on the simplified point array, not on raw input.

| Test | Method |
|---|---|
| **Closure** | Distance between first and last point, relative to the bounding-box diagonal |
| **Corner detection** | Local direction change above ~50° on the simplified polyline. Count and angular distribution classify rect (4 corners ~90°) vs triangle (3) vs diamond (4, rotated) |
| **Ellipse** | Ratio of stroke area to bounding-box area near π/4, with no dominant corners |
| **Line** | RDP with a large epsilon collapses to 2 points |
| **Arrow** | A line plus a short late direction reversal near one end |

The algorithm returns **ranked candidates**, not a single answer. This is what `Tab` cycles through, and it is why the same routine serves both the deliberate and the retroactive path.

### Snapping

The resulting primitive is snapped to the **stroke's bounding box**, not to a grid. The shape lands where you drew it, at the size you drew it.

`roundRect` corner radius is 8px at zoom 1 — the "presentable, smooth corners" form.

### Placing a shape directly

Shapes can also be placed from the [insert menu](Insert%20Menu.md) at a default size, skipping recognition entirely.

---

## States

A shape has no persisted states. `Tab` cycling is a runtime position in the candidate list:

| State | |
|---|---|
| **Cycling** | Consecutive `Tab` presses walk the ranked candidate list for one object. |
| **Settled** | Any other action ends the cycle. The next `Tab` starts a fresh cycle. |

Each step of the cycle is a command, so undo walks back through the cycle rather than jumping to the start.

---

## Constraints

- `rotation` is reserved and always `0`. Shapes cannot be rotated in v1 — see [Selection](Selection.md).
- Recognition operates on the simplified point array. Raw input is not retained past commit.
- The candidate cycle order is fixed and not configurable.
- Recognition never runs on its own. There is no automatic tidy, at any confidence level.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| `Tab` with nothing selected | Acts on the most recently created object. Whether it should instead act on whatever is under the cursor is unresolved — see TASKS `#4.1`. |
| `Tab` on an object that is not a stroke | Cycling on a selected [connector](Connectors.md) switches its routing, and `Tab` on a bound stroke converts it to a connector. Behavior on a text box, icon, or link card is not yet specified. |
| A stroke matches no candidate well | The algorithm ranks rather than thresholds, so a candidate is always returned. Whether a poor best-fit should be suppressed is not yet specified. |
| `Tab` cycling past `original` | Not yet specified. |
| A multi-object selection when `Tab` is pressed | Not yet specified. |
