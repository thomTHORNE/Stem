# Writing

## What it is

Write mode. Clicking places a text box and focuses it immediately. Text is rich — bold, italic, code, links, lists — and is rendered as an **HTML overlay positioned in screen space over the canvas**, not drawn to canvas.

Rasterizing text to canvas would break text selection and cost more than it saves. Only export rasterizes.

See [Data Model — TextBox](../Data%20Model.md#textbox).

---

## Behavior

### Creating a text box

- Click in Write mode places a text box at that point and focuses it.
- Width is set by dragging on creation, or defaults to **240px**. Height grows automatically.
- An empty text box is **discarded on blur** and never reaches the board file.

### Editing

- Double-clicking a text box in [Select mode](Selection.md) re-opens it for editing.
- `Esc` commits and returns to Select. Clicking outside also commits.

### Supported marks

Bold (`Cmd+B`), italic (`Cmd+I`), code (`Cmd+E`), strikethrough, link. Bullet and numbered lists.

**Nothing else** — no headings, no tables, no blockquotes. See [Design Principles](../Design%20Principles.md) → Constraint over configuration.

### Input rules

Markdown-style, applied as you type:

- `**bold**`
- `` `code` ``
- `- ` starts a list

### Rendering

While unfocused, the text box is **still an HTML element in the overlay**, transformed with the viewport. It is not rasterized and not moved to the scene canvas.

This is the one place where board content is not drawn by the renderer, and it is a deliberate exception: browser text layout is the reason `width` is the only dimension stored, and the reason height can grow without Stem computing anything.

### The text focus rule

While a text box has focus, **every single-letter mode key is inert.** Only `Esc` and `Cmd`-modified combinations pass through to the app.

This is stated in full in [Modes](Modes.md#the-text-focus-rule) and is the rule most likely to be broken by a careless keyboard handler.

---

## States

| State | |
|---|---|
| **Focused** | Being edited. Single-letter keys type. Vue's `isTextEditing` is true. |
| **Committed** | Unfocused, in the scene, still an HTML element in the overlay. |
| **Discarded** | Blurred while empty. Never persisted, never a command. |

The transition that matters is **Focused → Discarded**: an empty box leaves no trace, so a mis-click in Write mode costs nothing and needs no undo.

---

## Constraints

- Marks are limited to the list above. No headings, tables, or blockquotes.
- Only `width` is stored; height is derived from content by browser layout.
- Text boxes scale their **width only** — see [Selection](Selection.md).
- Text is never drawn to the canvas layers. Export is the only path that rasterizes it.
- One text box per creation click. There is no text flow between boxes.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| A text box is blurred while empty | Discarded. Never persisted. |
| A click in Write mode versus a drag to set width | Both create a box; drag sets width, click uses the 240px default. Where the threshold between the two gestures sits is not yet specified. |
| `Cmd`+`A` or `Cmd`+`Z` while focused | `Cmd`-modified combinations pass to the app, which collides with select-all-text and the editor's own undo. Not yet specified — see [Keybindings](Keybindings.md). |
| A text box is scaled | Width scales; height re-derives from the new width. |
| Content contains a mark outside the supported set, e.g. from a paste | Not yet specified. |
| A text box is exported | Rasterized to PNG, or emitted as `<foreignObject>` in SVG. See [Export](Export.md). |
