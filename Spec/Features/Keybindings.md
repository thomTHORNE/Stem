# Keybindings

## What it is

The complete key map. **This document is the source of truth for what a key does.** Feature specs describe behavior and reference a key by name; they never define one, and where a key appears in two features it is defined here once.

Keyboard-first is a design principle, not a convenience — see [Design Principles](../Design%20Principles.md). Every operation in Stem has a keyboard path, and the pointer is for making marks.

---

## Behavior

### Modes

Every mode key is spring-loaded. See [Modes](Modes.md) for tap-versus-hold semantics.

| Key | Mode |
|---|---|
| `Esc` / `V` | Select *(default, resting state)* |
| `D` | Draw |
| `W` | Write |
| `C` | Connect |
| `R` | Erase |

### Global

Active in every mode **except inside a focused text box**.

| Key | Action |
|---|---|
| `Esc` | Return to Select. Commit active text. Dismiss menu. Deselect. |
| `Space` (hold) | Pan, regardless of mode |
| `F` | Object hints — single letters overlay every visible object; press one to select it |
| `/` | [Insert menu](Insert%20Menu.md) at cursor |
| `Tab` | Tidy — convert the selected (or most recent) object to its clean form. Press again to cycle candidates. |
| `1`–`5` | Set color (applies to selection, or to the next thing drawn) |
| `Cmd`+`1`–`3` | Set stroke weight |
| `Cmd`+`A` | Select all |
| `Cmd`+`Z` / `Shift`+`Cmd`+`Z` | Undo / redo |
| `Cmd`+`C` / `Cmd`+`V` / `Cmd`+`D` | Copy / paste / duplicate |
| `Cmd`+`Shift`+`C` | Copy selection to clipboard as **PNG** |
| `Cmd`+`Alt`+`C` | Copy selection to clipboard as **SVG markup** |
| `Cmd`+`Shift`+`E` | Export selection (or whole board if nothing selected) to an `.svg` file |
| `Cmd`+`0` / `Shift`+`1` / `Shift`+`2` | Zoom 100% / fit all / fit selection |
| `Cmd`+`K` | Insert link (mention or bookmark) |
| `Cmd`+`N` | New board |
| `Cmd`+`O` | Board grid |
| `Cmd`+`,` | [Settings](Settings.md) |

### Selection

| Key | Action |
|---|---|
| `Delete` / `Backspace` | Delete selection |
| Arrow keys | Nudge 1px |
| `Shift`+arrows | Nudge 10px |
| `Cmd`+`Shift`+`+` / `-` | Scale selection about its center |

`Shift`+click adds to the selection. See [Selection](Selection.md).

### Text

Active only while a text box has focus. See [Writing](Writing.md).

| Key | Action |
|---|---|
| `Cmd`+`B` | Bold |
| `Cmd`+`I` | Italic |
| `Cmd`+`E` | Code |
| `Esc` | Commit and return to Select |

### Board grid

Active on the [board grid](Board%20Grid.md) start screen.

| Key | Action |
|---|---|
| Arrow keys | Navigate cards |
| `Enter` | Open |
| `F2` | Rename |
| `Delete` | Delete, with confirmation |
| `/` | Focus search-by-title |

### Hint mode

Entered by `F` (select an object) or `C` (connect two objects).

- Letters are assigned in **reading order** using a home-row-first alphabet: `asdfghjkl`, then `qwertyuiop`, then `zxcvbnm`.
- Two-letter labels once more than 26 objects are visible.
- `Esc` cancels.

Home-row-first is the reason hint mode is faster than pointing: the most common targets get the keys your fingers are already on.

---

## States

The key map is not flat — three states change what a key means:

| State | Effect on the map |
|---|---|
| **Normal** | The global map above applies. |
| **Text focused** | Every unmodified letter types a character. Only `Esc` and `Cmd`-modified combinations reach the app. |
| **Hint** | Letters select a hint target. `Esc` cancels. |

The partition is clean by construction: unmodified letters belong to modes and hints, `Cmd`-modified combinations belong to commands, and text takes the letters when it has focus.

---

## Constraints

- **Every unmodified letter is reserved** for modes and hint targets. No unmodified letter may be assigned to a command.
- Single-letter keys are inert inside a focused text box, without exception.
- The hint alphabet is fixed and not configurable.
- Keybindings are not user-remappable in v1.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| More than 26 objects visible in hint mode | Labels become two letters. How the two-letter labels are ordered and assigned is not yet specified. |
| `Cmd`+`A` inside a focused text box | `Cmd`-modified combinations pass through to the app, which would select all objects rather than all text. Not yet specified. |
| `Cmd`+`Z` inside a focused text box | Same collision: the app's undo stack and the text editor's own undo are both candidates. Not yet specified. |
| A hint letter is pressed that labels no visible object | Not yet specified. |
