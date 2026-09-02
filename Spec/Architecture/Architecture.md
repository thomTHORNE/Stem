# Architecture

This document records Stem's technical stack decisions and the reasoning behind them. It is the authoritative reference for technology choices.

The mechanics that follow from those choices are split out: [Rendering](Rendering.md), [Input](Input.md), and the [Command Layer](Command%20Layer.md). [Build Order](Build%20Order.md) sequences the work.

---

## Tech stack

| Layer | Choice | Notes |
|---|---|---|
| UI framework | Vue 3 + TypeScript | Composition API, `<script setup>` |
| Build | Vite | |
| Desktop shell | Electron | Added at the end; the app runs in a browser tab until then |
| Stroke geometry | `perfect-freehand` | Framework-agnostic, handles the full input→outline pipeline |
| Rich text | `Tiptap` v2 | First-class Vue 3 bindings |
| Hand-drawn primitives | `roughjs` (optional) | Only if the sketchy aesthetic is wanted on generated shapes |
| Spatial index | `rbush` (deferred) | Linear hit-testing is fine below ~2000 objects |

**Excalidraw and tldraw are not usable.** Both are React-only. Choosing Vue means owning the canvas engine. This is accepted, and it is the largest single consequence of the framework choice.

---

## Why Electron over Tauri

Tauri on macOS is WKWebView and on Windows is WebView2 — two different engines with different text metrics, dashed-line rendering, and clipboard behavior. For a canvas app shipped to both operating systems, one predictable Chromium is worth ~110MB of install size and ~120MB of idle RAM.

Text metrics and clipboard behavior are not incidental here. Text boxes are HTML in an overlay rather than rasterized to canvas ([Writing](../Features/Writing.md)), and export is three separate clipboard commands whose predictability is the whole point ([Export](../Features/Export.md)). Both are precisely the surfaces where two engines diverge.

### Mitigation for later regret

All filesystem, clipboard, and window access goes behind a `PlatformAdapter` interface, with a browser implementation and an Electron implementation. Swapping to Tauri later is then a contained job rather than a rewrite.

The adapter is not only an escape hatch — it is what lets the app run in a browser tab for the first ten steps of the [build order](Build%20Order.md), before the Electron shell exists. Capabilities the browser cannot provide return "unsupported" rather than failing; [Links](../Features/Links.md) is the case where that is user-visible.

### Electron config, day one

- `webPreferences.backgroundThrottling: false` — otherwise the render loop is throttled when the window is unfocused
- `contextIsolation: true`, `nodeIntegration: false`, all native access via `contextBridge`
- Keep vsync on; do not uncap frame rate

---

## The reactivity rule

**The scene graph must never be passed to `reactive()` or `ref()`.**

Vue's proxy-based reactivity deep-wraps every object it is given. Handed a stroke, it wraps the stroke's 400-point array, and the cost is then paid on every mutation and on every read during render — sixty times a second, for every point of every object on screen.

```
Scene (plain TS class)          ← owns all board data, no Vue involvement
  └─ Renderer (rAF loop)        ← reads Scene directly, draws to canvas
       └─ <canvas> element      ← Vue holds only a template ref to it

Vue reactive state              ← mode, activeColor, activeWeight, selectionCount,
                                   isTextEditing, menuOpen, boardTitle
```

Vue renders chrome: the mode indicator, the insert menu, the text editor overlay, the board grid. **It never renders board content.**

When the Scene changes something the chrome needs to know about, it emits on a small typed event bus and a composable updates a `ref`. The traffic across that boundary is deliberately thin — the list above is close to the whole of it.

---

## Deferred, conditional on measurement

These are not open questions. Each is a decision already made *not* to build something until a specific condition is observed.

| Deferred | Condition that would revisit it |
|---|---|
| Tile caching | Full scene redraw drops below 60fps. See [Rendering](Rendering.md). |
| `rbush` spatial index | Linear hit-testing becomes a bottleneck, expected above ~2000 objects. |
| `roughjs` | Only if the sketchy aesthetic is wanted on generated shapes. |

Building any of them earlier costs complexity against a problem that may never arrive.
