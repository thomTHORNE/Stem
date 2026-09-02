# Drawing

## What it is

Draw mode. Freehand ink follows the pointer and commits to a [Stroke](../Data%20Model.md#stroke) on release. It is the primary way marks get onto a board and the interaction everything else is arranged around.

Drawing is deliberately the one place in Stem where the pointer is the right tool. Everything else has a keyboard path.

---

## Behavior

### Stroke geometry

`perfect-freehand` handles the full input→outline pipeline. Its options are exposed in [Settings](Settings.md):

| Option | Default | Meaning |
|---|---|---|
| `size` | per weight slot: 2 / 4 / 8 | Base width |
| `thinning` | 0.5 | How much velocity affects width |
| `streamline` | 0.5 | Input smoothing — **the "line smoothing" knob** |
| `smoothing` | 0.5 | Outline curve smoothing |
| `simulatePressure` | true | Required; trackpad pressure is unavailable |
| `start.taper` / `end.taper` | 0 / 0 | Optional pen-lift taper |

`simulatePressure` is required rather than preferred. The MacBook Force Touch trackpad does not expose pressure to Chromium — see [Input](../Architecture/Input.md) → Pressure.

### While drawing

Wet ink is drawn on the overlay layer. The scene canvas is untouched until commit, so the cost of drawing does not grow with how much is already on the board. See [Rendering](../Architecture/Rendering.md).

Input arrives through the coalesced-events pipeline. Skipping that step produces visibly polygonal strokes on a trackpad.

### On commit (pointerup)

1. Run **Ramer–Douglas–Peucker** simplification, epsilon ≈ `0.6 / zoom`, reducing a 600-point raw stroke to roughly 40 points. Do this **in world space** so simplification is zoom-independent.
2. Compute and cache the AABB.
3. Run the endpoint binding check — see [Connectors](Connectors.md) → Path A.
4. Emit `AddStrokeCommand`.

The epsilon divides by zoom so that a stroke drawn while zoomed in is not simplified more aggressively than the same stroke drawn while zoomed out. The user's perceived precision is constant; the world-space tolerance is what has to move.

### What is stored

**The simplified point array, not the `perfect-freehand` outline.** The outline is regenerated at render time so stroke width can respond to zoom and to weight changes. See [Data Model — Stroke](../Data%20Model.md#stroke).

### Color and weight

- `1`–`5` set color, `Cmd`+`1`–`3` set weight. Both apply to the selection if there is one, otherwise to the next thing drawn.
- Both are stored as slots, never as resolved values. See [Visual Design](../Visual%20Design.md).

### Snapping to a primitive

Holding `Shift` before or during a stroke discards it on release and replaces it with its best-fit primitive. See [Shapes](Shapes.md).

---

## States

A stroke passes through two runtime states and then becomes an ordinary board object:

| State | |
|---|---|
| **Wet** | Between pointerdown and pointerup. Lives on the overlay layer, holds raw uncoalesced-plus-coalesced samples, is not in the scene, and is not undoable because it is not yet a command. |
| **Committed** | After pointerup. Simplified, AABB cached, binding resolved, added to the scene through a command. |

Nothing persists a wet stroke. A crash mid-stroke loses that stroke and nothing else.

---

## Constraints

- Pressure is unavailable. Width comes from velocity via `simulatePressure`, and no hardware path exists to change this.
- Five colors, three weights. There is no free-form width control and no color picker — see [Design Principles](../Design%20Principles.md) → Constraint over configuration.
- Simplification is lossy and runs on commit. The raw sample array is not retained.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| A stroke is a single point (tap with no movement) | Not yet specified. |
| `Shift` is released mid-stroke, before pointerup | The snap is evaluated on release. Whether the modifier state at pointerup or at any point during the stroke governs is not yet specified. |
| The pointer leaves the window mid-stroke | `setPointerCapture` keeps delivering the gesture, and `pointerup` arrives as expected. See [Input](../Architecture/Input.md). |
| A stroke is drawn at zoom 0.1 across a very large distance | Epsilon scales with zoom, so simplification tolerance in world space grows accordingly. No special handling. |
