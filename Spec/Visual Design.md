# Visual Design

Stem's visual language approximates Notion's, because Notion is where Stem's output lands. A diagram exported from Stem and pasted into a Notion page should not announce that it came from somewhere else.

This document holds the concrete values. [Design Principles](Design%20Principles.md) holds the reasoning that produced them.

---

## Chrome palette

```css
--bg:            #FFFFFF   /* dark: #191919 */
--bg-secondary:  #F7F6F3   /* dark: #202020 */
--border:        #E9E9E7   /* dark: #2F2F2F */
--text:          #37352F   /* dark: #D4D4D4 */
--text-muted:    #9B9A97   /* dark: #7F7F7F */
--accent:        #2383E2
--dot-grid:      #E1E1DF   /* dark: #2A2A2A */
```

`--accent` is the selection outline color and is the same in both themes.

---

## Ink color slots

Board content stores a **slot**, never a resolved color value. This is what allows an existing board to read correctly in both themes — the slot resolves at render time against the active theme.

| Slot | Light |
|---|---|
| `1` | default — `--text` |
| `2` | red — `#E03E3E` |
| `3` | blue — `#0B6E99` |
| `4` | green — `#0F7B6C` |
| `5` | yellow — `#DFAB01` |

Each slot has a dark-mode variant. The variant values are not yet defined.

Slot `1` follows `--text` and therefore inverts with the theme by construction. Slots `2`–`5` do not — a value chosen for legibility on `#FFFFFF` is not the value that works on `#191919`.

---

## Stroke weights

Three weights, stored as a slot on every object like color. Base widths are `2 / 4 / 8` px at zoom 1.

Ink scales with zoom: a weight-1 stroke at zoom 4 renders 8px wide. It is a mark on a canvas, not a UI element — see [Canvas & Viewport](Features/Canvas%20&%20Viewport.md).

---

## Type

`Inter`, then `ui-sans-serif`, then the system stack.

---

## Surfaces

- Border radius **3px** on chrome, **4px** on cards.
- Shadows are very restrained. `0 1px 2px rgba(0,0,0,0.06)` is usually enough.
- Dot grid: 20px spacing at zoom 1, in `--dot-grid`.

---

## Theme

Dark mode follows the OS, overridable in [Settings](Features/Settings.md).

Because board content stores slots rather than color values, switching themes reflows nothing and rewrites nothing. The board file is theme-independent.

---

## Chrome inventory

Chrome is minimal, and this is the complete list of what is permanently on screen:

- The [mode pill](Features/Modes.md), bottom-left — current mode, active color swatch, active weight.
- The board title, top-left.

Nothing else. Menus appear on invocation and vanish.
