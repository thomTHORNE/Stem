# Stem — v1 Specification

A keyboard-centric infinite canvas for thinking. Personal tool, Notion-adjacent.

---

## 1. Purpose and design principles

The tool exists to reproduce the one thing pen and paper does better than any software: **zero distance between a thought and its representation**. Everything below is subordinate to that.

**Principles**

1. **The tool must disappear.** If an action requires deciding *how* to do it, the design has failed. There is one way to draw a line, and it is: draw a line.
2. **No aiming.** Nothing important requires hitting a small target. Handles, grips, and resize dots are banned from the default interaction path. Whole objects are hit targets.
3. **Keyboard first, pointer second.** Every operation has a keyboard path. The pointer is for making marks, not for operating the software.
4. **Messy is a valid state.** The canvas never demands tidiness. Cleanup is an explicit, optional, reversible action the user invokes when a diagram has settled — never a thing the app does on its own.
5. **Constraint over configuration.** Five colors, three weights. No color picker, no line-style menu, no properties panel.

**Explicit non-goals for v1:** collaboration, mobile, layers, grouping, templates, presentation mode, image import, AI anything.

---

## 2. Stack

| Layer | Choice | Notes |
|---|---|---|
| UI framework | Vue 3 + TypeScript | Composition API, `<script setup>` |
| Build | Vite | |
| Desktop shell | Electron | Added at the end; app runs in a browser tab until then |
| Stroke geometry | `perfect-freehand` | Framework-agnostic, handles the full input→outline pipeline |
| Rich text | `Tiptap` v2 | First-class Vue 3 bindings |
| Hand-drawn primitives | `roughjs` (optional) | Only if the sketchy aesthetic is wanted on generated shapes |
| Spatial index | `rbush` (deferred) | Linear hit-testing is fine below ~2000 objects |

**Not usable:** Excalidraw and tldraw are React-only. Choosing Vue means owning the canvas engine. This is accepted.

### Why Electron over Tauri

Tauri on macOS is WKWebView and on Windows is WebView2 — two different engines with different text metrics, dashed-line rendering, and clipboard behavior. For a canvas app shipped to both OSes, one predictable Chromium is worth ~110MB and ~120MB of idle RAM.

Mitigation for later regret: all filesystem, clipboard, and window access goes behind a `PlatformAdapter` interface with a browser implementation and an Electron implementation. Swapping to Tauri later is then a contained job.

**Electron config, day one:**
- `webPreferences.backgroundThrottling: false` — otherwise the render loop is throttled when unfocused
- `contextIsolation: true`, `nodeIntegration: false`, all native access via `contextBridge`
- Keep vsync on; do not uncap frame rate

---

## 3. Architecture

### 3.1 The reactivity rule

**The scene graph must never be passed to `reactive()` or `ref()`.** Vue's proxy-based reactivity will deep-wrap every stroke object and its 400-point array, and the cost is paid on every mutation and every read during render.

```
Scene (plain TS class)          ← owns all board data, no Vue involvement
  └─ Renderer (rAF loop)        ← reads Scene directly, draws to canvas
       └─ <canvas> element      ← Vue holds only a template ref to it

Vue reactive state              ← mode, activeColor, activeWeight, selectionCount,
                                   isTextEditing, menuOpen, boardTitle
```

Vue renders chrome: mode indicator, insert menu, text editor overlay, board grid. It never renders board content. When the Scene changes something the chrome needs to know about, it emits on a small typed event bus and a composable updates a `ref`.

### 3.2 Rendering

Three stacked canvases, same size, absolutely positioned:

| Layer | Redraw trigger | Contents |
|---|---|---|
| **Background** | viewport change only | dot grid |
| **Scene** | scene mutation or viewport change | all committed objects |
| **Overlay** | every frame during interaction | wet ink, marquee, hint labels, selection outline |

Wet ink on its own layer is the single most important performance decision. While drawing, only the overlay redraws — the scene canvas is untouched.

- Get contexts with `{ desynchronized: true, alpha: true }`
- Handle `devicePixelRatio` explicitly; set canvas backing store to `cssSize * dpr` and scale the context
- Single `requestAnimationFrame` loop with a dirty-flag per layer; if nothing is dirty, do nothing
- **Deferred optimization:** if full scene redraw drops below 60fps, move to tile caching — bucket committed objects into 512×512 world-space tiles rendered to offscreen canvases, blit only visible tiles

### 3.3 Input pipeline

```
pointerdown/move/up (Pointer Events API, setPointerCapture)
  → getCoalescedEvents()        // trackpads report faster than the compositor ticks;
                                // skipping this drops samples and produces polygonal strokes
  → screen→world transform
  → per-mode handler
```

Pressure: the MacBook Force Touch trackpad's pressure is **not** exposed to Chromium via `PointerEvent.pressure` (it reports 0 or 0.5). Use `perfect-freehand`'s `simulatePressure: true`, which derives width from velocity. Fast strokes thin, slow strokes thicken. This reads as natural and is the only option available.

### 3.4 Command layer (undo/redo)

Every scene mutation is a command object:

```ts
interface Command {
  do(scene: Scene): void
  undo(scene: Scene): void
  label: string          // for debugging
  coalesceKey?: string   // consecutive commands with the same key merge
}
```

Two stacks, `undo` and `redo`. Redo clears on any new command.

`coalesceKey` handles continuous operations — a drag of one object emits many `MoveCommand`s that merge into one undo step, keyed on `move:${objectId}:${dragSessionId}`.

**Build this at step 3 of the build order, before there are many mutation types.** Retrofitting undo into thirty scattered call sites is the worst avoidable outcome in this project.

---

## 4. Data model

```ts
type ObjectId = string  // nanoid

interface BoardFile {
  version: 1
  id: ObjectId
  title: string
  createdAt: number
  updatedAt: number
  viewport: { x: number; y: number; zoom: number }  // restored on open
  objects: BoardObject[]
}

type BoardObject = Stroke | Shape | Connector | TextBox | Icon | LinkCard

interface BaseObject {
  id: ObjectId
  type: string
  bounds: Rect          // cached AABB, recomputed on mutation
  color: ColorSlot      // 1–5
  weight: WeightSlot    // 1–3
  z: number             // creation order; no layer UI in v1
}

interface Stroke extends BaseObject {
  type: 'stroke'
  points: [x: number, y: number][]   // simplified, world space
  binding: {
    from: ObjectId | null
    to: ObjectId | null
  }
  originalLength: number             // for the warp/reroute threshold
}

interface Shape extends BaseObject {
  type: 'shape'
  kind: 'rect' | 'roundRect' | 'ellipse' | 'triangle' | 'diamond' | 'line' | 'arrow'
  rect: Rect
  rotation: 0                         // reserved, not settable in v1
  fill: 'none' | 'solid'              // solid = background color, for occlusion
  label?: string                      // plain text, centered
}

interface Connector extends BaseObject {
  type: 'connector'
  from: { id: ObjectId } | { point: Point }
  to:   { id: ObjectId } | { point: Point }
  routing: 'curved' | 'orthogonal'
  arrowhead: 'end' | 'both' | 'none'
  label?: string
}

interface TextBox extends BaseObject {
  type: 'text'
  position: Point
  width: number                       // grows in height automatically
  content: JSONContent                // Tiptap document
}

interface Icon extends BaseObject {
  type: 'icon'
  setId: string
  iconId: string
  position: Point
  size: number
}

interface LinkCard extends BaseObject {
  type: 'link'
  url: string
  style: 'inline' | 'bookmark'
  title: string
  iconUrl?: string
  fetchedAt: number
}
```

One JSON file per board. No binary format, no database — the whole point is that a board is a small readable file.

---

## 5. Canvas and viewport

- **Infinite** in all directions. No boundaries, no page edges.
- **Dot grid** background, 20px spacing at zoom 1. Dots fade out below zoom 0.4 and switch to a coarser 100px grid below zoom 0.25.
- **Zoom range** 0.1 to 8. Zoom is centered on the cursor.
- **Trackpad gestures:** two-finger scroll pans; pinch zooms; `Cmd` + two-finger scroll also zooms. Never let the browser handle these — `preventDefault` on `wheel` with `passive: false`.
- **Ink scales with zoom.** A 2px stroke at zoom 4 renders 8px wide. It is a mark on a canvas, not a UI element.
- **Viewport commands:** `Cmd+0` reset to 100%, `Shift+1` zoom to fit all content, `Shift+2` zoom to fit selection.
- Viewport position is saved per board and restored on open.

---

## 6. Modes

Five modes. **Select is the resting state** and everything returns to it.

| Key | Mode | Behavior |
|---|---|---|
| `Esc` / `V` | **Select** *(default)* | Click anywhere on an object selects it. Click-drag on an object moves it. Drag on empty canvas draws a marquee. `Delete`/`Backspace` removes selection. Arrow keys nudge 1px, `Shift`+arrows nudge 10px. |
| `D` | **Draw** | Freehand ink follows the pointer. Hold `Shift` while drawing to snap the result to a primitive on release. |
| `W` | **Write** | Click places a text box at that point and focuses it immediately. |
| `C` | **Connect** | Hint letters appear on all visible objects. Type the source letter, then the target letter — a clean connector is generated. |
| `R` | **Erase** | Scrub-erase: swipe the pointer across objects to delete them. Deleting N objects in one swipe is one undo step. |

### 6.1 Spring-loading

Every mode key is spring-loaded:

- **Tap** (keydown → keyup with no pointer action in between, under 250ms): mode becomes sticky.
- **Hold** (keydown, then pointer action while held): the mode is active only until keyup, then snaps back to Select.

So: hold `R`, swipe across a bad stroke, release — you're back in Select with no bookkeeping. This is the feature that makes a modal tool feel fast. Implement it as a small state machine on keydown/keyup, not as a boolean.

### 6.2 The text focus rule

**While a text box has focus, every single-letter mode key is inert.** Only `Esc` and `Cmd`-modified combinations pass through to the app. `Esc` commits the text and returns to Select.

Get this wrong once and you will type a `d` into a paragraph and watch the app switch modes mid-sentence.

### 6.3 Mode indicator

A small persistent pill in the bottom-left showing the current mode, the active color swatch, and the active weight. It is the only chrome permanently on screen.

---

## 7. Keybindings

**Global — active in every mode except inside a focused text box**

| Key | Action |
|---|---|
| `Esc` | Return to Select. Commit active text. Dismiss menu. Deselect. |
| `Space` (hold) | Pan, regardless of mode |
| `F` | Object hints — single letters overlay every visible object; press one to select it |
| `/` | Insert menu at cursor |
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
| `Cmd`+`,` | Settings |

**Hint mode** (entered by `F` or `C`): letters are assigned in reading order using a home-row-first alphabet (`asdfghjkl` then `qwertyuiop` then `zxcvbnm`). Two-letter labels once more than 26 objects are visible. `Esc` cancels.

---

## 8. Drawing

`perfect-freehand` config, exposed in settings:

| Option | Default | Meaning |
|---|---|---|
| `size` | per weight slot: 2 / 4 / 8 | Base width |
| `thinning` | 0.5 | How much velocity affects width |
| `streamline` | 0.5 | Input smoothing — **the "line smoothing" knob** |
| `smoothing` | 0.5 | Outline curve smoothing |
| `simulatePressure` | true | Required; trackpad pressure is unavailable |
| `start.taper` / `end.taper` | 0 / 0 | Optional pen-lift taper |

**On commit** (pointerup):
1. Run Ramer–Douglas–Peucker simplification, epsilon ≈ 0.6 / zoom, to reduce a 600-point raw stroke to ~40 points. Do this in world space so simplification is zoom-independent.
2. Compute and cache the AABB.
3. Run endpoint binding check (§10.1).
4. Emit `AddStrokeCommand`.

Store the simplified point array, not the `perfect-freehand` outline. Regenerate the outline at render time so stroke width can respond to zoom and weight changes.

---

## 9. Shapes

Two ways to get a clean primitive from a hand-drawn stroke. Neither involves waiting.

### `Shift` while drawing — deliberate
Hold `Shift` before or during the stroke. On release the stroke is discarded and replaced by its best-fit primitive. Fully deterministic, zero false positives, zero latency. Use when you already know you want a clean box.

### `Tab` after drawing — retroactive
Press `Tab` to convert the selected object (or, if nothing is selected, the most recently created one) to its best-fit primitive. Press `Tab` again to cycle through the next-best candidates: `rect → roundRect → ellipse → diamond → triangle → line → arrow → original`.

This is the core of the "draw fast, tidy later" workflow. Because it's an ordinary command, `Cmd+Z` restores the original stroke.

### Recognition algorithm
On the simplified point array:
- **Closure**: distance between first and last point relative to bounding-box diagonal
- **Corner detection**: local direction change above ~50° on the simplified polyline; count and angular distribution classifies rect (4 corners ~90°) vs triangle (3) vs diamond (4, rotated)
- **Ellipse test**: ratio of stroke area to bounding-box area near π/4 with no dominant corners
- **Line test**: RDP with a large epsilon collapses to 2 points
- **Arrow test**: a line plus a short late direction reversal near one end

Ranked candidates are returned so `Tab` can cycle. Snap the resulting primitive to the stroke's bounding box, not to a grid.

`roundRect` corner radius: 8px at zoom 1. This is the "presentable, smooth corners" form.

---

## 10. Connectors — two paths, one purpose

Both produce a connection between objects. They differ in what the connection *looks like*, which is the point.

### 10.1 Path A — drawn, freehand

Draw a line in Draw mode. If an endpoint finishes within **24px** (screen space, so the radius is constant regardless of zoom) of an object's bounds, it binds silently. No handle, no aim, no mode, no visual affordance to hit. If the binding was unwanted, `Cmd+Z`.

**When a bound object moves — rigid warp.** Compute the delta at each bound endpoint and distribute it across the stroke's points, weighted by normalized arc-length distance from that endpoint:

```
weight(i) = 1 - (arcLengthFromBoundEnd(i) / totalArcLength)
point[i] += delta * weight(i)
```

The line stretches and leans but keeps every wobble you drew. Roughly 20 lines of code.

**Threshold:** if warping would take the stroke past **2× its `originalLength`**, fall back to regenerating a rough curve between the new anchors. In practice you nudge shapes a few hundred pixels and never hit this.

**On deletion of a bound object:** the stroke survives with that end unbound. Never cascade-delete.

### 10.2 Path B — generated, clean

`C` → hint letter on the source → hint letter on the target. A `Connector` object is created: a clean curved path with rounded corners between computed edge anchors, with an arrowhead.

Zero pointer movement. Two keystrokes to connect two objects anywhere on screen, including objects on opposite sides of a large board. On a trackpad this is faster than drawing, and it's the form you want once a diagram has settled.

**Pointer fallback:** in Connect mode, drag from anywhere *inside* the source to anywhere *inside* the target. The whole shape is the hit target. Handles are never rendered.

**Anchoring:** compute the line between the two objects' centers, intersect with each object's bounds, and attach there. On move, recompute — connectors always reroute, never warp.

**Routing:** `curved` by default (a cubic bézier with control points pushed out along the anchor normals). `orthogonal` available via `Tab` cycling on a selected connector, with 8px rounded corners.

### 10.3 Converting between them

`Tab` on a selected bound freehand stroke converts it to a clean `Connector` between the same two objects. `Tab` again reverts. This makes the whole tidy-up story one gesture: **`Tab` means "make this presentable."**

| | Path A (drawn) | Path B (generated) |
|---|---|---|
| Invoked by | drawing a line | `C` + two hint letters |
| Appearance | hand-drawn, keeps your wobble | clean curve, rounded corners |
| On move | rigid warp | reroute |
| Best for | thinking, exploring | settled diagrams, presentable output |

---

## 11. Writing

Click in Write mode places a text box and focuses it. Tiptap, rendered as an HTML overlay positioned in screen space over the canvas — not drawn to canvas.

**Supported marks:** bold (`Cmd+B`), italic (`Cmd+I`), code (`Cmd+E`), strikethrough, link. Bullet and numbered lists. Nothing else — no headings, no tables, no blockquotes.

- Width is set by dragging on creation, or defaults to 240px. Height grows automatically.
- Markdown-style input rules: `**bold**`, `` `code` ``, `- ` for a list.
- `Esc` commits and returns to Select. Clicking outside also commits.
- An empty text box is discarded on blur.
- Double-clicking a text box in Select mode re-opens it for editing.
- **Rendering:** while unfocused, the text box is still an HTML element in the overlay, transformed with the viewport. Rasterizing text to canvas would break selection and cost more than it saves. Only export rasterizes.

---

## 12. Insert menu (`/`)

Opens at the cursor position as a small filterable list. Type to filter, arrow keys or continued typing to narrow, `Enter` to place at the cursor, `Esc` to dismiss.

**Contents:**
- **Icons** — a curated set, searchable by name. Two sources: `lucide` for generic icons (~1400, MIT), `simple-icons` for software/brand logos (~3000, CC0). Both ship as raw SVG paths, so they render straight to canvas with no runtime dependency. Ship them as a pre-built index, not as a package import.
- **Shapes** — the primitives from §9, placed at a default size.
- **Link** — same as `Cmd+K`.
- **Frame** *(deferred to v2)*

Placed icons are single-color, taking the active color slot. Resize via `Cmd+Shift+` `+`/`-` on the selection; there are no resize handles.

---

## 13. Notion links (outbound only)

Two styles, matching Notion's own vocabulary:

- **Inline mention** — a small pill with the page icon and title, rendered on canvas
- **Bookmark** — a card with title, description, icon, and domain

`Cmd+K` opens a prompt. Two input paths:
1. **Paste a URL** — the app fetches the page and reads OpenGraph tags for title/description/icon. Works for any URL, not just Notion.
2. **Search Notion** — queries the Notion API `/v1/search` endpoint and lists matching pages to pick from. Requires an integration token.

Running inside Electron means no CORS restriction on either fetch — the main process makes the request. A pure browser build would need a proxy; the `PlatformAdapter` returns "unsupported" there.

**Token storage:** Notion integration token in Electron `safeStorage` (Keychain on macOS, DPAPI on Windows), configured in Settings. Never in the board file.

**Caching:** title and icon are cached in the `LinkCard` object with `fetchedAt`. Refresh on demand via a context action, not automatically.

**Clicking a link card** in Select mode opens the URL in the default browser — Notion desktop will intercept `notion.so` URLs itself.

---

## 14. Selection and manipulation

- **Click** an object selects it. `Shift`+click adds to selection.
- **Marquee** drag on empty canvas. Intersect-based, not fully-contained.
- **`F` hints** select without pointer movement.
- **Move:** drag any selected object, or arrow keys.
- **Delete:** `Delete` / `Backspace`, or Erase mode.
- **Resize:** `Cmd+Shift+` `+`/`-` scales the selection about its center. No handles. Shapes and icons scale; strokes scale their point arrays; text boxes scale their width only.
- **No rotation in v1.**
- **No grouping in v1.** Multi-select is transient.
- **Selection rendering:** a thin accent-colored outline around each selected object's bounds, drawn on the overlay layer. No corner dots.

---

## 15. Persistence and the board grid

### Storage
- One `.json` file per board in `<userData>/boards/`, filename `<id>.json`
- A `<userData>/index.json` holds `{ id, title, createdAt, updatedAt, thumbnailPath }` for fast grid rendering
- Thumbnails: 400×300 PNG rendered on save, in `<userData>/thumbnails/`
- The boards directory is user-configurable in Settings, so it can be pointed at iCloud/Dropbox

### Autosave and recovery
- Debounced save 800ms after the last mutation, plus a hard save every 30s during continuous activity
- **Atomic writes:** write to `<id>.json.tmp`, `fsync`, then rename. Never truncate the live file.
- Save on window blur and on `before-quit`
- A crash journal appends the last N commands to `<id>.journal`; on open, if a journal is newer than the board file, offer recovery

### Board grid (start screen)
- Vue-rendered, no canvas. Grid of thumbnail cards, newest-updated first.
- Card shows thumbnail, title, relative timestamp.
- Actions: open (click or `Enter`), rename (`F2` or double-click title), delete (`Delete`, with confirm), duplicate.
- Keyboard navigation with arrow keys; `/` focuses a search-by-title field.
- `Cmd+N` creates a board and opens it immediately with an untitled name.

---

## 16. Export

The governing insight: **Notion renders images at roughly 700px column width, does not render SVG inline, and has no panning.** A 4000px board pasted into Notion is an illegible smear.

Therefore the primary export is **the selection, not the board**. You explore on a sprawling canvas and copy out the one cluster that became a conclusion.

| Command | Output | Destination |
|---|---|---|
| `Cmd+Shift+C` | PNG, 2× DPI, cropped to selection bounds + 24px padding, transparent background | Clipboard as image — Notion pastes this |
| `Cmd+Alt+C` | SVG markup | Clipboard as plain text |
| `Cmd+Shift+E` | `.svg` file | Save dialog |

Separate commands rather than a single multi-format clipboard write, because when both an image and text are on the clipboard, the receiving app decides which to take and Notion's choice is not guaranteed. Predictability beats cleverness here.

**SVG generation:** strokes export as `<path>` from the `perfect-freehand` outline (filled, not stroked). Shapes and connectors export as native SVG elements. Text boxes export as `<foreignObject>` with inline HTML — with a fallback flag in Settings to convert text to paths for compatibility with tools that ignore `foreignObject`.

If nothing is selected, export covers the whole board's content bounds.

---

## 17. Visual design

Notion's language, approximated:

```css
--bg:            #FFFFFF   /* dark: #191919 */
--bg-secondary:  #F7F6F3   /* dark: #202020 */
--border:        #E9E9E7   /* dark: #2F2F2F */
--text:          #37352F   /* dark: #D4D4D4 */
--text-muted:    #9B9A97   /* dark: #7F7F7F */
--accent:        #2383E2
--dot-grid:      #E1E1DF   /* dark: #2A2A2A */
```

Ink color slots `1`–`5`: default (`--text`), red `#E03E3E`, blue `#0B6E99`, green `#0F7B6C`, yellow `#DFAB01`. Each slot has a dark-mode variant.

Type: `Inter`, then `ui-sans-serif`, then system stack. Border radius 3px on chrome, 4px on cards. Shadows very restrained — `0 1px 2px rgba(0,0,0,0.06)` is usually enough.

Dark mode follows the OS, overridable in Settings. Board content colors switch by slot, so an existing board reads correctly in both themes.

Chrome is minimal: the mode pill bottom-left, a board title top-left, nothing else. Menus appear on invocation and vanish.

---

## 18. Build order

Sequenced so the riskiest work comes first and every later step is additive.

| # | Step | Notes |
|---|---|---|
| 1 | **Canvas core** | Three-layer canvas, infinite dot grid, pan/zoom, `Scene` class, rAF loop, coalesced pointer events, DPR handling. No UI. |
| 2 | **Draw mode** | `perfect-freehand`, RDP simplify, commit, colors, weights. *Use it on real work for a few days before continuing.* |
| 3 | **Command layer** | Undo/redo, coalescing. Do this before there are many mutation types. |
| 4 | **Select mode** | Hit testing, marquee, move, delete, nudge, `F` hints. |
| 5 | **Persistence** | Board file format, atomic autosave, crash journal, board grid start screen. |
| 6 | **Write mode** | Tiptap overlay. Largest single chunk of work. |
| 7 | **Export** | PNG clipboard, SVG clipboard, SVG file. |
| 8 | **Shapes** | Recognition algorithm, `Shift`-snap, `Tab`-convert with cycling. |
| 9 | **Connectors** | Proximity binding, rigid warp, `Connector` type, `C` hints, `Tab` conversion between forms. |
| 10 | **Insert menu** | `/` menu, icon index, link cards, Notion search. |
| 11 | **Electron shell** | `PlatformAdapter` Electron implementation, native menus, safeStorage, packaging. |

Steps 1–5 produce something genuinely usable. Everything from 6 onward can be reordered by whatever is annoying you most that week.

---

## 19. Open questions

1. **Board storage location** — default to `<userData>/boards/`, or ask for a folder on first run so it can sit in iCloud from day one?
2. **Notion API access** — do you want the search-based mention picker (needs an integration token, needs pages shared with the integration), or is paste-a-URL enough for v1? The paste path is ~2 hours; the search path is ~a day plus setup friction.
3. **Erase granularity** — does scrub-erase delete a whole stroke on contact, or split the stroke at the erased segment? Whole-stroke is far simpler and matches how you'd use a pen eraser on a diagram; segment-splitting matches a real eraser but complicates the data model.
4. **Shape fill** — do you want the `solid` fill option (shape gets the background color so it occludes what's behind it), or is everything transparent outline? Occlusion matters once diagrams overlap.
5. **Shape labels** — should shapes carry a centered text label directly, or do you always place a separate text box on top? A direct label means the text moves with the shape, which matters a lot for diagrams.
6. **Weight slots** — are 2/4/8px the right three? Easy to change, but worth picking deliberately since there's no free-form width control.
7. **`Tab` scope** — when nothing is selected, `Tab` acts on the most recently created object. Should it instead act on whatever is under the cursor? The latter is more direct but requires the pointer to be somewhere meaningful.
8. **Cmd+K link placement** — does an inserted link land at the cursor, or at the center of the viewport?
