# Design Principles

Stem exists to reproduce the one thing pen and paper does better than any software: **zero distance between a thought and its representation.** Everything in this spec is subordinate to that.

These five principles govern every feature — interaction, layout, defaults, and what is refused. They are not aesthetic preferences. They are the reason the tool is shaped the way it is. If a proposed element or interaction cannot be justified by at least one of them, it does not belong in Stem.

---

## The tool must disappear

If an action requires deciding *how* to do it, the design has failed. There is one way to draw a line, and it is: draw a line.

A decision the user has to make about the software is a decision they are not making about their work. The cost is not the time the decision takes — it is that the thought they were holding is gone by the time they come back.

> In practice: there is no line-style menu, because there is no choice to make. Hold `Shift` and the stroke snaps clean; don't and it stays as drawn.

---

## No aiming

Nothing important requires hitting a small target. Handles, grips, and resize dots are banned from the default interaction path. **Whole objects are hit targets.**

Aiming is the tax modal drawing tools charge on every operation, and it is paid with the hand rather than the mind — which is why it breaks a train of thought so reliably.

> In practice: connectors bind by proximity, not by grabbing a handle. In Connect mode, the whole shape is the drop target. Resizing is a keystroke on the selection, not a corner drag. `cached bounds` on every object is what makes this affordable — see [Data Model](Data%20Model.md) → BaseObject.

---

## Keyboard first, pointer second

Every operation has a keyboard path. The pointer is for making marks, not for operating the software.

Hint letters exist so that selecting and connecting are keyboard operations even though the objects live at arbitrary positions on an infinite plane. Two keystrokes connect two objects on opposite sides of a large board — no pointer travel, no scrolling to bring both into reach.

> In practice: [Keybindings](Features/Keybindings.md) is the source of truth for what every key does, and no feature spec redefines one.

---

## Messy is a valid state

The canvas never demands tidiness. Cleanup is an explicit, optional, reversible action the user invokes when a diagram has settled — never something the app does on its own.

Software that tidies as you go is software that interrupts. A hand-drawn box that stays a hand-drawn box until you press `Tab` is a box that never argued with you mid-thought.

> In practice: `Tab` means "make this presentable," and it is always the user pressing it. Recognition never fires on a timer, on release, or on a confidence threshold. And because tidying is an ordinary command, `Cmd+Z` puts the wobble back.

---

## Constraint over configuration

Five colors, three weights. No color picker, no line-style menu, no properties panel.

A constrained palette is not a limitation to work around — it is what makes a board legible weeks later, and what removes a decision from every single mark.

> In practice: color and weight are stored as slots rather than resolved values, which is also what lets an existing board read correctly in both light and dark themes. See [Visual Design](Visual%20Design.md).

---

## v1 non-goals

These are excluded from v1 deliberately. They are not oversights, and none of them is blocked on the others.

| Excluded | |
|---|---|
| Collaboration | Stem is a personal tool. No multi-user model, no presence, no shared state. |
| Mobile | Desktop only. The interaction model is a trackpad and a full keyboard. |
| Layers | `z` is creation order and there is no UI for it. |
| Grouping | Multi-select is transient and is not persisted. |
| Templates | Every board starts empty. |
| Presentation mode | Output leaves via [export](Features/Export.md), into a document that already has a reader. |
| Image import | No raster content on the canvas. |
| AI anything | |

Excluding these is what makes the rest of v1 small enough to finish. Anything here worth reconsidering later lives in [Ideas.md](../Ideas.md), not in this table.
