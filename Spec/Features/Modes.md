# Modes

## What it is

Stem is modal. Five modes, each holding one job, with **Select as the resting state** that everything returns to.

Modality is usually the thing that makes a tool feel slow — you end up doing bookkeeping about what state the app is in instead of working. Spring-loading is what removes that cost: a mode you hold is a mode you cannot forget you are in, because releasing the key ends it.

See [Keybindings](Keybindings.md) for the complete key table. This document owns mode semantics; it does not redefine keys.

---

## Behavior

### The five modes

| Key | Mode | Behavior |
|---|---|---|
| `Esc` / `V` | **Select** *(default)* | Click anywhere on an object selects it. Click-drag on an object moves it. Drag on empty canvas draws a marquee. `Delete`/`Backspace` removes the selection. Arrow keys nudge 1px, `Shift`+arrows nudge 10px. |
| `D` | **Draw** | Freehand ink follows the pointer. Hold `Shift` while drawing to snap the result to a primitive on release. |
| `W` | **Write** | Click places a text box at that point and focuses it immediately. |
| `C` | **Connect** | Hint letters appear on all visible objects. Type the source letter, then the target letter — a clean connector is generated. |
| `R` | **Erase** | Scrub-erase: swipe the pointer across objects to delete them. Deleting N objects in one swipe is one undo step. |

Each mode has its own document: [Selection](Selection.md), [Drawing](Drawing.md), [Writing](Writing.md), [Connectors](Connectors.md).

### Spring-loading

Every mode key is spring-loaded. The distinction is made by what happens between keydown and keyup:

| Gesture | Result |
|---|---|
| **Tap** — keydown → keyup with no pointer action in between, under 250ms | Mode becomes **sticky**. It stays until changed. |
| **Hold** — keydown, then a pointer action while held | Mode is active only until keyup, then **snaps back to Select**. |

So: hold `R`, swipe across a bad stroke, release — you are back in Select with no bookkeeping. This is the feature that makes a modal tool feel fast.

**Implement it as a small state machine on keydown/keyup, not as a boolean.** The states are distinguished by history, not by a single current value: "held, no pointer action yet" and "held, pointer action seen" behave differently on keyup and cannot be told apart by one flag.

### The text focus rule

**While a text box has focus, every single-letter mode key is inert.** Only `Esc` and `Cmd`-modified combinations pass through to the app.

`Esc` commits the text and returns to Select.

Get this wrong once and you will type a `d` into a paragraph and watch the app switch modes mid-sentence.

This rule is why [Keybindings](Keybindings.md) reserves every unmodified letter for modes and hints: those keys are unavailable inside text by design, so the rule is a clean partition rather than a list of exceptions.

### Mode indicator

A small persistent pill in the bottom-left showing:

- the current mode
- the active color swatch
- the active weight

It is the only chrome permanently on screen besides the board title. See [Visual Design](../Visual%20Design.md) → Chrome inventory.

---

## States

The mode system is runtime state — nothing here is persisted, and a reopened board starts in Select.

| State | Entered by | Left by |
|---|---|---|
| **Sticky** | Tapping a mode key | Tapping another mode key, or `Esc` |
| **Held** | Holding a mode key | Releasing the key — returns to Select |
| **Text focused** | A text box taking focus | `Esc`, or clicking outside |
| **Hint** | `F` or `C` | Selecting a target, or `Esc` |

**Hint** and **Text focused** both suspend the normal keyboard mapping while they are active, and both exit on `Esc`. They are the only two states in Stem that do this.

---

## Constraints

- Select is the resting state. Every other mode returns to it.
- Hold-mode always returns to Select on keyup — never to the mode that was sticky beforehand.
- Single-letter mode keys are unavailable inside a focused text box, without exception.
- There is no mode that persists across board open. Modes are session state.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| A mode key is held and released with no pointer action, over 250ms | The tap threshold is 250ms, and the gesture is defined by whether a pointer action occurred. A slow tap with no pointer action is not yet specified. |
| A second mode key is pressed while the first is held | Not yet specified. |
| A mode key is held while a text box has focus | Single-letter keys are inert while text has focus, so the key types a character. Spring-loading does not engage. |
| `Esc` pressed during a held mode | Not yet specified. |
| The window loses focus mid-hold — keyup never arrives | Not yet specified. |
