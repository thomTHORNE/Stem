# Build Order

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

---

## Where the sequence is load-bearing

Three of these orderings are decisions rather than convenience, and moving them has consequences:

**Step 2 before step 3.** Draw mode comes first so there is something real to use before the architecture around it hardens. The instruction to use it on real work for a few days is part of the step, not a suggestion — it is the only point in the plan where the core interaction can still be cheaply wrong.

**Step 3 before everything after it.** The [command layer](Command%20Layer.md) is placed here because it is the one piece that cannot be retrofitted cheaply. Steps 4 onward each add mutation types; every one of them written before the command layer exists is a call site that has to be found again.

**Step 11 last.** The app runs in a browser tab until then, behind the `PlatformAdapter` from [Architecture](Architecture.md). Electron is packaging, not foundation.

---

## Where it is flexible

**Steps 1–5 produce something genuinely usable** — a canvas you can draw on, select on, undo on, and that keeps your work. That is the milestone worth defending.

Everything from step 6 onward can be reordered by whatever is annoying you most that week. The dependencies among them are weak: shapes and connectors both benefit from select mode existing, and the insert menu wants shapes to exist, but nothing later blocks anything earlier.
