# Selection

## What it is

Select is the resting state — the mode everything returns to. It is where objects are picked, moved, scaled, and deleted.

Its defining constraint is that **there are no handles.** Whole objects are hit targets, selection is a thin outline with no corner dots, and every manipulation has a keyboard path. See [Design Principles](../Design%20Principles.md) → No aiming.

---

## Behavior

### Selecting

- **Click** an object to select it. `Shift`+click adds to the selection.
- **Marquee** — drag on empty canvas. **Intersect-based, not fully-contained**: an object that the marquee touches is selected.
- **`F` hints** select without pointer movement. Letters overlay every visible object; press one. See [Keybindings](Keybindings.md) → Hint mode.
- `Cmd`+`A` selects all.

Intersect-based marquee suits a canvas where objects are large and sprawling — requiring full containment would mean zooming out just to lasso a diagram you can already see.

### Moving

- Drag any selected object.
- Arrow keys nudge 1px; `Shift`+arrows nudge 10px.

A drag emits many move commands that coalesce into one undo step. See [Command Layer](../Architecture/Command%20Layer.md).

### Scaling

`Cmd`+`Shift`+`+` / `-` scales the selection **about its center**. There are no resize handles.

| Object | Scales |
|---|---|
| Shapes | `rect` |
| Icons | `size` |
| Strokes | their point arrays |
| Text boxes | **width only** — height re-derives from content |

### Deleting

`Delete` / `Backspace`, or [Erase mode](Modes.md).

Deleting an object that a stroke or connector is bound to unbinds that end. Nothing cascade-deletes — see [Connectors](Connectors.md).

### Rendering

A thin `--accent` outline around each selected object's bounds, drawn on the **overlay layer**. No corner dots.

Because the outline is on the overlay, changing the selection costs one overlay redraw and never touches committed content. See [Rendering](../Architecture/Rendering.md).

---

## States

Selection is runtime state and is not persisted. A reopened board has nothing selected.

| State | |
|---|---|
| **Empty** | Nothing selected. `Tab` falls back to the most recently created object; export falls back to the whole board. |
| **Single** | One object. `Tab` and scaling act on it. |
| **Multi** | More than one. Transient — there is no grouping in v1, and the selection is lost as soon as it is changed. |

The **Empty** state is load-bearing rather than trivial: several commands are defined by what they do when there is no selection, which is what lets them be pressed without first pointing at something.

---

## Constraints

- **No handles**, in any mode, for any object type.
- **No rotation in v1.** `Shape.rotation` is reserved and always `0`.
- **No grouping in v1.** Multi-select is transient and never persisted.
- No z-order controls. `z` is creation order — see [Data Model](../Data%20Model.md#baseobject).
- Selection is never persisted to the board file.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| A selected object is deleted while bound to a stroke | The stroke survives with that end unbound. |
| `Shift`+drag on empty canvas | `Shift`+click adds to the selection; whether `Shift`+marquee adds to it is not yet specified. |
| Scaling a multi-object selection | Scales about the selection's center. How per-object scaling composes across mixed types is not yet specified. |
| Scaling a stroke | The point array scales. Whether stored `weight` also scales, or only the geometry, is not yet specified. |
| Marquee drag that touches nothing | Selection becomes empty. |
| An object is selected and the mode changes | Not yet specified. |
