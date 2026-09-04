# Tasks

Stem spec writing tracker. Structure: Alignment → Goal → Item.

## Status

- `[ ]` **Unresolved** — default state. No spec work drafted yet.
- `[x]` **Pending review** — spec work drafted and committed, awaiting user review.

Resolved tasks are removed from TASKS.md.

## Severity

- `BLOCKER` — missing spec that would produce broken or undefined behavior at implementation time.
- `GAP` — incomplete or inconsistent spec that would cause a developer to make a wrong assumption.
- `MINOR` — should be addressed but low risk if deferred; implementation can proceed without it.

Items marked **§19** were carried across from the original spec's own open questions. Everything else surfaced while splitting it.



## Alignment 1 — Foundation

::: toggle `#1` Data Model
The object schema is the source of truth for the board file format. Anything undefined here is undefined in the file on disk.

[Spec/Data Model.md](Spec/Data%20Model.md)
- - -
- [ ] `#1.1` `BLOCKER` **`Rect` and `Point` field shapes are undefined**
    Both are referenced by `BaseObject.bounds`, `Shape.rect`, `TextBox.position`, `Icon.position`, and `Connector`'s free-point ends, and neither is declared. `Rect` has at least three plausible shapes — `{x, y, w, h}`, `{x1, y1, x2, y2}`, `{minX, minY, maxX, maxY}` — and hit-testing, marquee intersection, connector anchoring, and export cropping all read it. Two of them being written against different assumptions is a class of bug that will not surface until diagrams overlap.
<br>
- [ ] `#1.2` `BLOCKER` **`ColorSlot` and `WeightSlot` types are undefined**
    The ranges are stated as comments (`1–5`, `1–3`) but neither type is declared. Whether they are numeric literal unions, an enum, or bare `number` determines whether an out-of-range slot is caught at compile time or renders as nothing at all.
<br>
- [ ] `#1.3` `GAP` **`LinkCard` has no `description` field**
    The bookmark style renders "title, description, icon, and domain", but `LinkCard` carries only `url`, `style`, `title`, `iconUrl`, and `fetchedAt`. The fetch reads a description from OpenGraph and then has nowhere to put it. Either the field is missing or the bookmark style does not render one.
<br>
- [ ] `#1.4` `GAP` **`Stroke.binding` scope is undefined**
    `binding` sits on every `Stroke`, not only line-like ones, but the binding rule is written as "draw a line in Draw mode". Whether a scribbled note ending near a box binds to it — and then warps when the box moves — is unstated. Warping a non-line stroke would be visibly wrong.
<br>
- [ ] `#1.5` `GAP` **`Connector` free-point ends are unreachable**
    `from` and `to` accept `{ point: Point }` as well as `{ id: ObjectId }`, but every specified path — `C` plus two hints, and the pointer fallback — produces two bound ends. No interaction creates a free-point connector. Either the case is dead and comes out of the type, or the interaction that reaches it is missing.
<br>
- [ ] `#1.6` `GAP` **`color` has no defined meaning on `TextBox` and `LinkCard`**
    `BaseObject` puts `color` and `weight` on every object. For strokes, shapes, connectors, and icons both are obvious. For a text box, whether `color` tints the text, and what `weight` means at all, is unstated — and a link card renders fetched content whose colors are not Stem's to choose.
<br>
- [ ] `#1.7` `GAP` `Deps: #1.4` **Shape `fill` — keep or drop** `§19`
    `fill: 'none' | 'solid'` gives a shape the background color so it occludes what is behind it. Occlusion matters once diagrams overlap. Decide whether v1 carries it or everything is transparent outline. This is not only a data-model question — it propagates to export, where the PNG background is transparent and `solid` has nothing to resolve against (`#9.1`).
<br>
- [ ] `#1.8` `GAP` **Shape `label` — direct label or separate text box** `§19`
    A shape can carry a centered plain-text `label`. The alternative is always placing a `TextBox` on top. A direct label moves with the shape, which matters a lot for diagrams; a separate box is rich text and is one fewer text-rendering path to build. Both are currently in the spec — `label` on `Shape`, and `TextBox` as a first-class object.
<br>
- [ ] `#1.9` `MINOR` **`z` behavior across `Tab` conversion is undefined**
    `Tab` replaces a stroke with a shape, and a bound stroke with a connector. Whether the replacement inherits the original's `z` or takes the next one is unstated. Inheriting preserves the picture; taking the next one silently brings tidied objects to the front.
:::

::: toggle `#2` Architecture
Structural decisions and the interfaces that carry them.

[Spec/Architecture/Architecture.md](Spec/Architecture/Architecture.md)
- - -
- [ ] `#2.1` `GAP` **`PlatformAdapter` interface is named but not defined**
    It is the boundary for all filesystem, clipboard, and window access, the reason the app can run in a browser tab before the Electron shell exists, and the stated mitigation for choosing Electron over Tauri. Its actual surface is nowhere. Every capability it fronts is specified from the other side — atomic writes, three clipboard formats, a save dialog, `safeStorage` — so the interface is derivable, but until it is written the browser and Electron builds have no contract to share.
<br>
- [ ] `#2.2` `GAP` **Scene → Vue event bus contract is not enumerated**
    The reactive state Vue holds is listed — mode, active color, active weight, selection count, `isTextEditing`, menu open, board title — but the events that update it are described only as "a small typed event bus". The list of events is the whole width of the boundary between the scene graph and the framework, and keeping it thin is the entire point of the reactivity rule.
:::



## Alignment 2 — Canvas & Interaction

::: toggle `#3` Modes & Keybindings
Spring-loading is what makes a modal tool bearable, and it is a state machine with several unstated transitions.

[Spec/Features/Modes.md](Spec/Features/Modes.md) · [Spec/Features/Keybindings.md](Spec/Features/Keybindings.md)
- - -
- [ ] `#3.1` `BLOCKER` **Spring-load transitions are only half-defined**
    Tap and hold are defined; four transitions out of them are not. A second mode key pressed while the first is held; `Esc` pressed during a held mode; a slow press over 250ms with no pointer action; and the window losing focus mid-hold so `keyup` never arrives. The last one leaves the app stuck in a mode the user is no longer holding, which is the exact failure spring-loading exists to prevent.
<br>
- [ ] `#3.2` `BLOCKER` **`Cmd`-modified keys collide inside a focused text box**
    The text focus rule passes every `Cmd`-modified combination through to the app. `Cmd+A` therefore selects all objects rather than all text, and `Cmd+Z` targets the app's undo stack rather than the editor's. Both are reflexes a user will fire inside a paragraph. The rule needs an exception list, or the text editor needs first claim on a named set.
<br>
- [ ] `#3.3` `GAP` **Erase granularity** `§19`
    Does scrub-erase delete a whole stroke on contact, or split the stroke at the erased segment? Whole-stroke is far simpler and matches how a pen eraser is used on a diagram. Segment-splitting matches a real eraser and complicates the data model — a split produces two strokes from one, each needing its own `bounds`, `originalLength`, and binding resolution.
<br>
- [ ] `#3.4` `MINOR` **Two-letter hint labels are unspecified**
    Labels become two letters once more than 26 objects are visible. The assignment order, whether the first letter is still reading-order, and whether a one-letter label can prefix a two-letter one are all unstated. The last matters: if `a` labels an object and `as` labels another, the first keystroke is ambiguous.
<br>
- [ ] `#3.5` `MINOR` **A hint keystroke matching no visible object**
    Whether it cancels hint mode, is ignored, or beeps. Cheap to decide, and it is the most common way a hint interaction goes wrong.
:::

::: toggle `#4` Drawing & Shapes
Recognition and the `Tab` cycle. The interaction is well specified; its boundaries are not.

[Spec/Features/Drawing.md](Spec/Features/Drawing.md) · [Spec/Features/Shapes.md](Spec/Features/Shapes.md)
- - -
- [ ] `#4.1` `GAP` **`Tab` scope when nothing is selected** `§19`
    `Tab` currently acts on the most recently created object. Should it instead act on whatever is under the cursor? The latter is more direct but requires the pointer to be somewhere meaningful — which conflicts with keyboard-first, where the pointer is often nowhere in particular.
<br>
- [ ] `#4.2` `GAP` **`Tab` on object types that have no clean form**
    Cycling is defined for strokes and for connectors. `Tab` on a text box, icon, or link card is unstated. "Make this presentable" has no meaning for an icon, so the answer is probably that it does nothing — but a no-op that is never stated reads as a bug.
<br>
- [ ] `#4.3` `GAP` **`Tab` on a multi-object selection**
    Whether it tidies each object independently, does nothing, or acts on one of them. Tidying a whole diagram in one keystroke is either the best thing in the app or the most destructive, and which it is depends entirely on this answer.
<br>
- [ ] `#4.4` `GAP` **Modifier state governing a `Shift`-snap**
    `Shift` may be held before or during a stroke, and the snap is evaluated on release. Whether the modifier state at `pointerup` governs, or whether `Shift` held at any point during the stroke is enough, is unstated. Releasing `Shift` early is a common slip.
<br>
- [ ] `#4.5` `GAP` **A single-point stroke**
    A tap in Draw mode with no movement. RDP on one point, an AABB of zero area, and a hit target of nothing. Whether it produces a dot or is discarded.
<br>
- [ ] `#4.6` `MINOR` **Weight slot values** `§19`
    Are `2 / 4 / 8` px the right three? Easy to change, but worth picking deliberately since there is no free-form width control and these three are the entire vocabulary.
<br>
- [ ] `#4.7` `MINOR` **`Tab` cycling past `original`**
    The candidate list ends at `original`. Whether the next press wraps to `rect` or the cycle stops there.
<br>
- [ ] `#4.8` `MINOR` **Poor best-fit suppression**
    Recognition ranks rather than thresholds, so a candidate is always returned — including for a stroke that resembles no primitive at all. Whether a weak best-fit should be suppressed, and against what measure.
<br>
- [ ] `#4.9` `GAP` **Recognition against a zero-area bounding box**
    The ellipse test is the ratio of stroke area to bounding-box area. A perfectly horizontal or vertical stroke has a bounding box of zero area, so that ratio divides by zero and the candidate ranks as `NaN` rather than ranking low. `NaN` fails every comparison it appears in, so where the ellipse lands in the ranked list is decided by the sort implementation rather than by the algorithm — `Tab` on a straight line can offer an ellipse, with nothing having errored. Wider than `#4.5`: a flat stroke is not a single point and is not a candidate for discarding.
:::

::: toggle `#5` Connectors
Both binding paths work as specified. What is missing is what happens when the target is ambiguous.

[Spec/Features/Connectors.md](Spec/Features/Connectors.md)
- - -
- [ ] `#5.1` `GAP` `Deps: #6.1` **A stroke endpoint within 24px of two objects**
    Overlapping objects, or two objects closer together than the binding radius. Nothing says which wins. This resolves the same way hit-testing does, and answering it separately risks two different rules for "which object is at this point".
<br>
- [ ] `#5.2` `GAP` **Source and target being the same object in Connect mode**
    Typing the same hint letter twice, or dragging from inside an object back into it. A self-connector has no anchor pair — the line between an object's center and itself has no direction to intersect its bounds with.
:::

::: toggle `#6` Selection
Hit-testing underpins selection, connector binding, erase, and the pointer fallback in Connect mode. It is specified in none of them.

[Spec/Features/Selection.md](Spec/Features/Selection.md)
- - -
- [ ] `#6.1` `BLOCKER` **Hit-testing method is undefined**
    "Click anywhere on an object selects it" and "whole objects are hit targets" both read as bounds-based, but a stroke's AABB is mostly empty space — a large diagonal stroke would claim a rectangle covering half the diagram. Whether hit-testing uses cached bounds, actual geometry, or bounds as a broad phase followed by a geometry test is unstated, as is which object wins when several are hit. Selection, scrub-erase, connector binding, and the Connect-mode pointer fallback all depend on the answer.
<br>
- [ ] `#6.2` `GAP` **`Shift`+marquee**
    `Shift`+click adds to the selection. Whether `Shift`+drag adds a marquee's contents to the selection or replaces it is unstated, and additive marquee is how any multi-object edit across a large board gets assembled.
<br>
- [ ] `#6.3` `GAP` **Scaling a stroke — geometry only, or weight too**
    A stroke scales its point array. Whether the stored `weight` slot also scales is unstated. It cannot scale continuously — there are only three slots — so either the geometry grows while the ink stays the same thickness, or the slot steps and the scale is lossy.
<br>
- [ ] `#6.4` `GAP` **Scaling a mixed multi-object selection**
    Scaling is about the selection's center, and each type scales differently — shapes their `rect`, icons their `size`, strokes their points, text boxes their width only. How those compose so the selection scales as one unit, particularly when a text box's height re-derives from its new width, is unstated.
<br>
- [ ] `#6.5` `MINOR` **Whether selection survives a mode change**
    Tapping `D` with three objects selected. Keeping the selection makes `1`–`5` recolor it; dropping it makes them set the next-drawn color. Both are defensible, and the mode pill shows the active color either way.
:::



## Alignment 3 — Content

::: toggle `#7` Writing
The text overlay is the largest single chunk of work in the build order and the least specified at its edges.

[Spec/Features/Writing.md](Spec/Features/Writing.md)
- - -
- [ ] `#7.1` `GAP` **Click versus drag on text box creation**
    A click uses the 240px default width; a drag sets the width. Where the threshold between the two gestures sits is unstated, and every click carries a few pixels of movement.
<br>
- [ ] `#7.2` `GAP` **Paste carrying unsupported marks**
    The supported set is bold, italic, code, strikethrough, link, and two list types. Pasting from a browser or a Notion page will carry headings, tables, and colors. Whether they are stripped, flattened, or rejected determines whether `TextBox.content` can be trusted to hold only what the spec allows.
:::

::: toggle `#8` Insert Menu & Links
[Spec/Features/Insert Menu.md](Spec/Features/Insert%20Menu.md) · [Spec/Features/Links.md](Spec/Features/Links.md)
- - -
- [ ] `#8.1` `GAP` **Fetch failure, offline, and missing OpenGraph tags**
    Three failure paths with no defined behavior, on the one feature in Stem that touches the network. A URL with no OG tags is the common case, not the exception — plenty of pages have none, and the card then has no title to render.
<br>
- [ ] `#8.2` `GAP` **What `Cmd+K` does in the browser build**
    The `PlatformAdapter` returns "unsupported". Whether the prompt is hidden, disabled, or shown with an explanation is unstated — and this is the only user-visible gap between the browser build and the shipped app during ten of the eleven build steps.
<br>
- [ ] `#8.3` `GAP` **Where an inserted link lands** `§19`
    At the cursor, or at the center of the viewport. The insert menu places at the cursor, so `Cmd+K` landing elsewhere would be the only placement in Stem that does not.
<br>
- [ ] `#8.4` `MINOR` **Icon set attribution**
    `lucide` is MIT, which requires the licence and copyright notice to be distributed with the software. `simple-icons` is CC0 and requires nothing, but its brand marks carry trademark obligations that a licence does not cover. Shipping a pre-built index rather than the packages does not remove either.
<br>
- [ ] `#8.5` `MINOR` **Insert menu with no results, and `Enter` with nothing highlighted**
<br>
- [ ] `#8.6` `MINOR` **Insert menu opened near a viewport edge**
    It anchors at the cursor and has nowhere to render.
:::



## Alignment 4 — Output & Storage

::: toggle `#9` Export
Export is the point of the tool — the board is where you think, the export is what you keep.

[Spec/Features/Export.md](Spec/Features/Export.md)
- - -
- [ ] `#9.1` `GAP` `Deps: #1.7` **`fill: 'solid'` against a transparent PNG background**
    Solid fill is the background color, and the PNG exports with a transparent background. The fill has nothing to resolve against: rendered as the theme's background it becomes an opaque block in a transparent image; omitted, the occlusion the shape was drawn for disappears. Blocked on whether `fill` survives at all.
<br>
- [ ] `#9.2` `GAP` **Which theme an export resolves against**
    Ink slots resolve per theme. A board exported in dark mode and pasted into a light Notion page carries dark-mode ink. Whether export always resolves light, follows the active theme, or is a setting.
<br>
- [ ] `#9.3` `MINOR` **Export with no selection and no content**
    Export falls back to the whole board's content bounds, and an empty board has none.
<br>
- [ ] `#9.4` `MINOR` **An export larger than the receiving app accepts**
    Selection-first export exists precisely because a whole board is unusable in Notion, but nothing stops a user selecting all and pressing `Cmd+Shift+C` on a 4000px board.
:::

::: toggle `#10` Persistence
Atomic writes and the journal are well specified. What is not specified is everything about the directory the files live in.

[Spec/Features/Persistence.md](Spec/Features/Persistence.md)
- - -
- [ ] `#10.1` `BLOCKER` **Concurrent writes when the boards directory is a sync folder**
    The directory is user-configurable so it can sit in iCloud or Dropbox, which is stated as a feature. Two machines with Stem open on the same board then both autosave every 800ms into a folder that resolves conflicts by filename. Atomic rename protects a single writer from a crash; it does nothing against a second writer. Either the sync case is supported and needs a rule, or it is stated as unsupported.
<br>
- [ ] `#10.2` `GAP` **Board storage location on first run** `§19`
    Default to `<userData>/boards/`, or ask for a folder on first run so it can sit in iCloud from day one? Asking on first run puts a filesystem decision in front of a user who wanted to draw. Defaulting means the boards move later, which is `#10.4`.
<br>
- [ ] `#10.3` `GAP` **Journal depth `N` is undefined**
    The crash journal appends "the last N commands". The value determines what recovery can actually recover — too small and the journal covers less than the gap between saves it exists to bridge.
<br>
- [ ] `#10.4` `GAP` **Changing the boards directory while boards exist**
    Whether existing boards move, are copied, or are left behind, and what the index does about it.
<br>
- [ ] `#10.5` `GAP` **`index.json` disagreeing with `boards/`**
    The index is derived and the board files are authoritative, which says which one wins but not what happens. A board file present but unindexed is invisible in the grid; an indexed board with no file is a card that opens nothing. Both arise from an interrupted write or an external file operation, which a sync folder makes ordinary.
<br>
- [ ] `#10.6` `GAP` **A board file that fails to parse, or carries an unknown `version`**
    `version: 1` exists so a future format change can be detected rather than guessed at, and nothing says what detecting one does.
<br>
- [ ] `#10.7` `GAP` **Recovery declined**
    Whether the journal is discarded, kept, or offered again on next open.
<br>
- [ ] `#10.8` `MINOR` **Thumbnail rendering cost against the save cadence**
    A 400×300 PNG is rendered on save, and saves fire 800ms after the last mutation plus every 30s during continuous activity. Whether a thumbnail is rendered on every one of those, or on a slower cadence.
<br>
- [ ] `#10.9` `MINOR` **Viewport save cadence**
    `viewport` is persisted per board, and panning mutates it continuously. Whether a viewport change marks the board dirty like any other mutation.
:::

::: toggle `#11` Board Grid
[Spec/Features/Board Grid.md](Spec/Features/Board%20Grid.md)
- - -
- [ ] `#11.1` `GAP` **First-run empty state**
    No boards exist. This is the first screen anyone ever sees in Stem, and it is unspecified.
<br>
- [ ] `#11.2` `GAP` **Single-letter shortcuts while renaming or searching**
    Both states capture typing, exactly as a focused text box does on the canvas. The text focus rule covers the canvas and says nothing about this screen.
<br>
- [ ] `#11.3` `MINOR` **Delete — remove the file or move it to trash**
    Delete is the only action in Stem that asks for confirmation, and a recoverable delete is a different promise from an unrecoverable one.
<br>
- [ ] `#11.4` `MINOR` **A duplicated board's title**
<br>
- [ ] `#11.5` `MINOR` **A missing or unrenderable thumbnail**
:::



## Alignment 5 — Shell & Integrations

::: toggle `#12` Settings
[Spec/Features/Settings.md](Spec/Features/Settings.md)
- - -
- [ ] `#12.1` `BLOCKER` **The Settings surface is unspecified**
    Six things are committed to living in Settings — the boards directory, the dark-mode override, the `perfect-freehand` options, the `foreignObject` export fallback, and the Notion token — and the surface itself has no spec. `Cmd+,` opens something with no defined shape, navigation, or persistence. The inventory exists; the screen does not.
<br>
- [ ] `#12.2` `GAP` **`perfect-freehand` options changed after strokes exist**
    Strokes store points and regenerate their outline at render time, so changing `size`, `thinning`, or `smoothing` retroactively changes every stroke on every board. That may be the intent — it is the same mechanism that lets ink scale with zoom — but it means a settings slider silently rewrites the appearance of past work.
<br>
- [ ] `#12.3` `MINOR` **Settings in the browser build**
    Three of its entries need the Electron shell, and `safeStorage` does not exist there at all.
:::

::: toggle `#13` Notion
[Spec/Integrations/Notion.md](Spec/Integrations/Notion.md)
- - -
- [ ] `#13.1` `GAP` **Whether the search path ships in v1** `§19`
    The search-based mention picker needs an integration token and pages explicitly shared with the integration; paste-a-URL needs neither. The paste path is roughly two hours of work, the search path roughly a day plus setup friction — and the friction lands on the user, once, before the feature does anything.
<br>
- [ ] `#13.2` `MINOR` **A revoked or invalid token**
<br>
- [ ] `#13.3` `MINOR` **Search with no token configured**
    Whether the search path is hidden, disabled, or offers to configure one.
:::



## Alignment 6 — Visual Design

::: toggle `#14` Visual Design
[Spec/Visual Design.md](Spec/Visual%20Design.md)
- - -
- [ ] `#14.1` `BLOCKER` **Dark-mode ink slot values are not defined**
    "Each slot has a dark-mode variant" is stated and no variant is given. Slot `1` follows `--text` and inverts by construction, but slots `2`–`5` are fixed hex values chosen against `#FFFFFF` — `#0B6E99` blue and `#0F7B6C` green are both close to unreadable on `#191919`. Storing slots rather than resolved colors is the mechanism that makes theme-independent boards possible, and it does nothing until the second set of values exists.
:::



## Ideas — deferred, not in active scope

See [Ideas.md](Ideas.md).
