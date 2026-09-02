# Insert Menu

## What it is

A small filterable list opened with `/` at the cursor position. It is the single entry point for content that is placed rather than drawn — icons, shapes, and links.

Type to filter, arrow keys or continued typing to narrow, `Enter` to place at the cursor, `Esc` to dismiss.

---

## Behavior

### Contents

| Entry | |
|---|---|
| **Icons** | A curated set, searchable by name. |
| **Shapes** | The primitives from [Shapes](Shapes.md), placed at a default size. |
| **Link** | Same as `Cmd+K`. See [Links](Links.md). |
| **Frame** | *Deferred to v2.* |

### Icon sources

Two sets ship with the app:

| Set | Count | Licence | For |
|---|---|---|---|
| `lucide` | ~1400 | MIT | Generic icons |
| `simple-icons` | ~3000 | CC0 | Software and brand logos |

Both ship as **raw SVG paths**, so they render straight to canvas with no runtime dependency.

**Ship them as a pre-built index, not as a package import.** Importing either package pulls a component library into a canvas app that has no use for components — the paths are the only part Stem needs.

### Placement

- `Enter` places the entry at the cursor.
- Placed icons are **single-color**, taking the active color slot.
- Resize via `Cmd`+`Shift`+`+`/`-` on the selection. There are no resize handles — see [Selection](Selection.md).

Single-color icons are not a simplification of the sets; it is the same rule that governs every other mark on the board. A brand logo renders in ink like everything else.

---

## States

The menu is transient runtime state:

| State | |
|---|---|
| **Closed** | Default. `/` opens it. |
| **Open** | Anchored at the cursor, unfiltered. |
| **Filtering** | A query has been typed; the list is narrowed. |

`Esc` dismisses from either open state and returns to the current mode.

---

## Constraints

- The icon set is curated and fixed. There is no icon import in v1.
- Icons are single-color.
- Frames are deferred to v2 — see [Ideas.md](../../Ideas.md).
- The menu opens at the cursor, not at a fixed position.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| `/` pressed while a text box has focus | Single-letter keys are inert while text has focus, so `/` types a slash. |
| A filter query matches nothing | Not yet specified. |
| `Enter` with no entry highlighted | Not yet specified. |
| The menu is opened near a viewport edge with no room to render | Not yet specified. |
