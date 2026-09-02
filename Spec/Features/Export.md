# Export

## What it is

Getting a diagram out of Stem and into a document.

The governing insight: **Notion renders images at roughly 700px column width, does not render SVG inline, and has no panning.** A 4000px board pasted into Notion is an illegible smear.

Therefore the primary export is **the selection, not the board.** You explore on a sprawling canvas and copy out the one cluster that became a conclusion.

---

## Behavior

### The three commands

| Command | Output | Destination |
|---|---|---|
| `Cmd+Shift+C` | PNG, 2× DPI, cropped to selection bounds + 24px padding, transparent background | Clipboard as image — Notion pastes this |
| `Cmd+Alt+C` | SVG markup | Clipboard as plain text |
| `Cmd+Shift+E` | `.svg` file | Save dialog |

If nothing is selected, export covers the **whole board's content bounds**.

### Why three commands, not one

Separate commands rather than a single multi-format clipboard write, because when both an image and text are on the clipboard, **the receiving app decides which to take** and Notion's choice is not guaranteed.

Predictability beats cleverness here. A user who pressed the PNG key and got markup pasted has no way to tell what went wrong.

### SVG generation

| Object | Exported as |
|---|---|
| Strokes | `<path>` from the `perfect-freehand` outline — **filled, not stroked** |
| Shapes, connectors | Native SVG elements |
| Text boxes | `<foreignObject>` with inline HTML |

Strokes export as filled paths because that is what they are: `perfect-freehand` produces an outline enclosing the stroke, not a centerline with a width. Exporting it as a stroked path would render a hollow shape.

**`foreignObject` fallback:** a flag in [Settings](Settings.md) converts text to paths, for compatibility with tools that ignore `foreignObject`.

---

## States

Export has no persisted state. It is a read of the scene at the moment the command fires.

Its behavior is defined by the [selection](Selection.md) state at that moment, which is the only input it takes besides the board:

| Selection | Export covers |
|---|---|
| **Empty** | The whole board's content bounds |
| **Single or multi** | The selection's bounds, plus 24px padding for PNG |

---

## Constraints

- PNG is 2× DPI, always. There is no export resolution setting.
- PNG padding is 24px, fixed.
- Export never includes the dot grid, the mode pill, or any chrome.
- There is no PDF export, no multi-board export, and no export presets.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| Export with an empty board and no selection | There are no content bounds to crop to. Not yet specified. |
| A shape with `fill: 'solid'` exported to PNG | `solid` fill is the background color and the PNG background is transparent. What the fill renders as is not yet specified. |
| Export while in dark mode | Ink slots resolve differently per theme. Which theme an export resolves against is not yet specified. |
| A text box exported to SVG with the fallback flag off, opened in a tool that ignores `foreignObject` | The text does not render. This is what the fallback flag exists for. |
| An export larger than the receiving app can handle | Not yet specified. |
