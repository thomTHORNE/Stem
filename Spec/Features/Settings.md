# Settings

## What it is

The configuration surface, opened with `Cmd+,`.

Stem is built on **constraint over configuration** — five colors, three weights, no properties panel — so Settings is deliberately small. What it holds is not a menu of preferences but the short list of things the spec has committed to being adjustable, mostly because they are environment-dependent rather than taste-dependent.

**This document is an inventory, not a specification of the surface.** Each entry below is referenced from the feature that needs it. How Settings is presented, navigated, or persisted is not yet specified.

---

## Behavior

### What Settings holds

| Setting | Referenced from | |
|---|---|---|
| **Boards directory** | [Persistence](Persistence.md) | Where board files, the index, and thumbnails live. Configurable so the directory can sit in iCloud or Dropbox. |
| **Dark mode** | [Visual Design](../Visual%20Design.md) | Follows the OS by default; overridable here. |
| **`perfect-freehand` options** | [Drawing](Drawing.md) | `size`, `thinning`, `streamline`, `smoothing`, `simulatePressure`, `start.taper` / `end.taper`. `streamline` is the "line smoothing" knob. |
| **`foreignObject` fallback** | [Export](Export.md) | Converts text to paths on SVG export, for tools that ignore `foreignObject`. |
| **Notion integration token** | [Integrations — Notion](../Integrations/Notion.md) | Stored in Electron `safeStorage`, never in a board file. |

### What Settings deliberately does not hold

Colors, weights, keybindings, grid spacing, and zoom limits are all fixed. Each is a constraint the design depends on rather than a default awaiting adjustment — see [Design Principles](../Design%20Principles.md).

---

## States

Not yet specified.

---

## Constraints

- The Notion token is stored in the OS secure enclave (Keychain on macOS, DPAPI on Windows) via Electron `safeStorage`. It is never written to a board file and never to plain settings storage.
- Keybindings are not remappable — see [Keybindings](Keybindings.md).
- Settings that require the Electron shell are unavailable in the browser build.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| Settings opened in the browser build, where `safeStorage` does not exist | Not yet specified. |
| `perfect-freehand` options changed after strokes have been drawn | Strokes store points and regenerate their outline at render time, so a changed option affects existing strokes. Whether that is intended is not yet specified. |
| The boards directory is changed while boards exist | Not yet specified — see [Persistence](Persistence.md). |
