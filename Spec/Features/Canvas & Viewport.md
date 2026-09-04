# Canvas & Viewport

## What it is

The surface everything else happens on: an infinite plane with a dot grid, panned and zoomed by trackpad or keyboard. It has no boundaries and no page edges, so a board never runs out of room and the user never has to decide where something goes in relation to an edge that does not exist.

The viewport is the window onto that plane. Its position and zoom are board state — saved with the board and restored on open, so reopening a board returns you to where you were looking.

See [Data Model — BoardFile](../Data%20Model.md#boardfile) for the persisted `viewport` field, and [Rendering](../Architecture/Rendering.md) for how the background layer is drawn.

---

## Behavior

### The plane

- **Infinite** in all directions. No boundaries, no page edges, no content bounds the user can hit.
- Object positions are world-space coordinates. Screen position is a function of the viewport and nothing else.

### Dot grid

- 20px spacing at zoom 1, drawn in `--dot-grid`. See [Visual Design](../Visual%20Design.md).
- Dots **fade out below zoom 0.4**.
- Below **zoom 0.25**, the grid switches to a coarser 100px spacing.

The grid is a sense of scale, not a snapping target. Nothing snaps to it — shape recognition snaps a primitive to the source stroke's bounding box, never to the grid.

### Zoom

- Range **0.1 to 8**.
- Zoom is **centered on the cursor**, not on the viewport center. The point under the pointer stays under the pointer.

### Ink scales with zoom

A 2px stroke at zoom 4 renders 8px wide. Ink is a mark on a canvas, not a UI element, and it behaves like one — zooming in on a diagram magnifies the drawing rather than revealing thinner lines.

This is why strokes store a simplified point array and regenerate their outline at render time. See [Data Model — Stroke](../Data%20Model.md#stroke).

### Panning

- Two-finger trackpad scroll pans, in every mode.
- Holding `Space` pans by drag, regardless of mode.

### Trackpad gestures

- Two-finger scroll pans; pinch zooms; `Cmd` + two-finger scroll also zooms.
- The browser must never handle these — see [Input](../Architecture/Input.md) → Trackpad gestures.

### Viewport commands

| Key | Action |
|---|---|
| `Cmd+0` | Reset to 100% |
| `Shift+1` | Zoom to fit all content |
| `Shift+2` | Zoom to fit selection |

[Keybindings](Keybindings.md) is the source of truth for these.

No viewport command is undoable. Pan and zoom are not scene mutations and emit no command — see [Command Layer](../Architecture/Command%20Layer.md). `Cmd+0`, `Shift+1` and `Shift+2` are therefore one-way: the way back to a previous view is another viewport command, not `Cmd+Z`.

---

## States

The viewport has no persisted states beyond its position and zoom. It has two runtime states worth naming, because they change what redraws:

| State | |
|---|---|
| **Settled** | No viewport change in progress. Background and scene layers are clean; nothing redraws. |
| **Transforming** | A pan or zoom is in progress. Background and scene layers are both dirty every frame. |

Unlike drawing or dragging — which touch only the overlay — a viewport change is the one interaction that dirties all three layers at once. See [Rendering](../Architecture/Rendering.md).

---

## Constraints

- Zoom is clamped to `0.1`–`8`.
- The grid is a visual reference only. There is no snap-to-grid in v1, and no grid configuration.
- There is no minimap, no scrollbars, and no "you are here" indicator. `Shift+1` is the way back to the content.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| `Shift+1` (fit all) on an empty board | There is no content to fit. Not yet specified. |
| `Shift+2` (fit selection) with nothing selected | Not yet specified. |
| Zoom to fit content that exceeds the zoom range in one axis | Not yet specified. |
