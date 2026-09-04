# Bounds & Rectangles

General canvas ground, with Stem as the running example. Not spec — nothing here defines Stem's behavior, and where this document and `Spec/` disagree, the spec is right and this is out of date.

Written to give the vocabulary needed for the `Rect` and `Point` field-shape decision, and for the hit-testing decision that follows it. Neither decision is made here. §3 and §6 each lay out a tradeoff and stop.

New words are defined where they first appear and collected in [Glossary](Glossary.md).

---

## 1 · World space and screen space

You draw a stroke. Then you zoom to 4× and pan halfway across the board.

The stroke is now somewhere else on your display. Its pixels have moved. **Has the stroke changed?**

The answer has to be no — you didn't touch it, you just looked somewhere else. But that answer only holds if the stroke was never described in terms of your display in the first place.

### What happens if you store display pixels

Suppose each object stored where it currently sits on screen. Panning is now a document edit. Every pan frame rewrites the coordinates of every object on the board:

- 200 objects, 60 frames a second, is **12,000 coordinate rewrites per second** — of your actual saved document, not of some render cache.
- Every one of those is a mutation, and in Stem every mutation goes through the command layer. Your undo history fills with scrolling. `Cmd+Z` walks back through where you were looking.
- Each rewrite adds a pan delta to the previous result. Floating-point arithmetic loses a little precision every time, so panning back and forth for an hour leaves your drawing measurably deformed. The board decays from being *looked at*.
- The saved file encodes where you happened to be looking when you saved.

None of that is a bug you fix. It is what storing display pixels means.

### Two coordinate systems

So you use two, and only ever store one of them.

**World space** is the infinite canvas itself. It has no viewer, no zoom, and no edges — Stem's plane is infinite in all directions, so a world coordinate like `(4820, -1130)` is a perfectly ordinary place. This is what a board file contains.

**Screen space** is pixels on your actual display. Nothing stores it. It is computed fresh every frame from world space plus the viewport, and thrown away.

The **viewport** is what connects them: `{ x, y, zoom }`, saved per board. It is not a property of any object — it is a property of *your view of* the objects.

### The conversion

Going from world to screen: subtract where the viewport is, then multiply by zoom.

```
A stroke's corner sits at world (100, 100).

viewport { x: 0,  y: 0,  zoom: 1 }   →  screen (100, 100)
viewport { x: 50, y: 20, zoom: 2 }   →  screen ((100 − 50) × 2, (100 − 20) × 2)
                                     =  screen (100, 160)
viewport { x: 50, y: 20, zoom: 4 }   →  screen (200, 320)
```

The world coordinate `(100, 100)` never changed. It was `(100, 100)` in all three.

Going the other way — screen to world — is the same steps undone: divide by zoom, then add the viewport back. You need this constantly, because **the pointer arrives in screen space and everything it is tested against is in world space.**

```
Pointer lands at screen (300, 200), viewport { x: 50, y: 20, zoom: 2 }

world x = 300 ÷ 2 + 50 = 200
world y = 200 ÷ 2 + 20 = 120
```

That conversion is not a formality. It is the first line of every hit test in the app, and forgetting it produces a bug where clicking works perfectly at 100% zoom and misses by an increasing margin the further you zoom.

The exact convention Stem uses — the order of operations, where the origin sits — belongs in `Spec/Architecture/Rendering.md` and gets settled when the canvas core is built. The shape of it is what matters here.

### When screen space is the right answer

Storing screen space is wrong. *Thinking* in screen space is sometimes exactly right, and Stem already does it deliberately in one place.

A stroke binds to an object when its endpoint finishes **within 24px, screen space**, of that object's bounds.

Screen space, not world. That means the threshold is "as close as it looks to you" — the same visual gap binds whether you are zoomed in or out. Had it been 24 world units, the rule would tighten as you zoom in and become impossibly loose zoomed out, and the feature would feel unpredictable for reasons you'd never guess from the outside.

The rule: **positions are stored in world space; perceptual thresholds are expressed in screen space.** Anything about where a thing *is* is world. Anything about how close something *looks* is screen.

---

## 2 · Bounding boxes

Start with a real cost. You've drawn a stroke and you want to drag it, so Stem has to answer: **is the pointer on this stroke?**

Answering honestly means testing the pointer against every segment of the stroke — the line from point 1 to point 2, point 2 to point 3, and so on. Stem simplifies a raw 600-point stroke down to roughly 40 on commit, so that's about 40 distance calculations. For one stroke.

A board with 200 objects on it is 8,000 calculations. And this doesn't run once when you click — it runs on **every pointer move**, so while you're dragging a marquee across the board it runs 60 times a second. Nearly half a million calculations a second, to answer a question that is *no* for almost every object almost every time.

So you don't ask the honest question first.

### The cheap question

Say you drew a long diagonal stroke that wanders from around `(100, 100)` down to `(500, 400)`. Whatever it does in between, every point on it sits inside this rectangle:

```
        left 100                       right 500
            ┌──────────────────────────────┐
  top 100   │ ●───╮                        │
            │      ╰──╮                    │
            │          ╰───╮               │
            │               ╰──╮           │
            │        ✕          ╰──╮       │
 bottom 400 └──────────────────────●───────┘
                 (300, 380)
```

That's the **bounding box**, and `bounds` on `BaseObject` is where Stem caches one for every object.

Now the pointer arrives at `(600, 250)`. Two comparisons:

```
is 600 left of the box's left edge (100)?    no
is 600 right of the box's right edge (500)?  YES → eliminated
```

Two comparisons and the stroke is gone. Not "probably not on it" — *certainly* not on it, because a point outside the box is outside everything the box contains. Forty distance calculations skipped, and the answer is provably identical to the honest one.

Do that across 200 objects and you've replaced 8,000 expensive tests with a few hundred trivial ones, leaving maybe two or three objects worth asking properly about. Cheap test over everything, then the expensive test over the survivors, is called **broad phase** then **narrow phase**. `bounds` is Stem's broad phase.

### Where the cheap question lies to you

Look at `✕` on the diagram — the pointer at `(300, 380)`.

```
is 300 within 100–500?   yes
is 380 within 100–400?   yes  → the box says HIT
```

But the ink at `x = 300` is up around `y = 250`. The pointer is **130 pixels** away from the nearest ink. The box said yes and the box was wrong.

This isn't a flaw in the technique, it's the deal you signed. That box is 400 × 300 — 120,000 square units of claim — and the actual ink is a thin line through it. A diagonal stroke's box is almost entirely empty space, and it claims all of it.

Which is exactly why hit-testing is a separate blocker on the tracker. The broad phase says *maybe*, and something still has to say *actually*. If Stem never builds the narrow phase, clicking blank canvas near a big diagonal stroke selects the stroke — and to you that reads as the app grabbing things you didn't point at.

### Why it never tilts

**Axis-aligned** — the AABB in the Data Model's comment — means the box's edges stay parallel to the x and y axes.

A tilted box would hug that diagonal stroke far more tightly and waste much less space. The catch is the test. Checking a point against an upright box is the two comparisons above. Checking it against a tilted one means first rotating the point into the box's own frame — trigonometry, per object, per pointer move. You'd trade a few hundred comparisons for 200 rotations *plus* a few hundred comparisons, to save work you were going to do in the narrow phase anyway.

Looser and free beats tighter and expensive when the whole job is elimination.

### Why it's a cache, not the truth

The Data Model already says `bounds` is a cache and the geometry wins. The example says why: **the points are the stroke.** The box is only a claim *about* the points, and the moment you drag the stroke or scale it, that claim is false until it's recomputed.

A stale box is a stroke you can't click — the pointer is on the ink, the box is somewhere the ink used to be, the broad phase eliminates it, and the narrow phase never runs. The object is visibly there and completely dead. That failure is nearly impossible to spot in code review and obvious the instant you use the app.

---

## 3 · The three ways to write a rectangle down

You've decided every object caches a box. Now you have to write one into a JSON file.

A rectangle needs four numbers. The question is **which four**, and it has three common answers that describe the identical box.

Here is one box — left edge 100, top 100, right 500, bottom 400 — in all three:

```
A   { x: 100, y: 100, w: 400, h: 300 }              origin and size
B   { x1: 100, y1: 100, x2: 500, y2: 400 }          two corners
C   { minX: 100, minY: 100, maxX: 500, maxY: 400 }  extents
```

**B and C hold the same four numbers.** The difference is the promise. `min`/`max` is a claim that the first is never larger than the second, so anything reading it can rely on that. `x1`/`x2` is just "two corners in the order given" and promises nothing — which turns out to matter in §6, so hold onto it.

That leaves the real contest between **A** and **C**, and it's decided by which operations get easier.

### Operation 1 — building one from a stroke

You have 40 points. You need the box.

```
C:   minX = smallest x,  maxX = largest x        ← that's the whole job
A:   same work, then w = maxX − minX, h = maxY − minY
```

C is what the loop naturally produces. A needs a conversion step at the end.

### Operation 2 — the marquee overlap test

The one that runs 60 times a second while you drag. Does the stroke's box overlap the marquee's box?

```
stroke  left 100, top 100, right 500, bottom 400
marquee left 450, top 350, right 700, bottom 600
```

They overlap — in the corner region from `450–500` across and `350–400` down. Both forms get that right; they don't read the same:

```
C:  stroke.maxX >= marquee.minX  &&  stroke.minX <= marquee.maxX
 &&  stroke.maxY >= marquee.minY  &&  stroke.minY <= marquee.maxY

A:  stroke.x + stroke.w >= marquee.x  &&  stroke.x <= marquee.x + marquee.w
 &&  stroke.y + stroke.h >= marquee.y  &&  stroke.y <= marquee.y + marquee.h
```

Same logic. But C reads as four plain comparisons, and A has four additions threaded through it that must land on the correct side of each comparison. Getting one of them backwards gives you a marquee that selects things it didn't touch, or misses things it did — and it works fine in most cases, which is what makes it hard to find.

### Operation 3 — union

Export with nothing selected covers the whole board's content bounds. `Shift+1` fits all content. Both mean: combine many boxes into the one box containing all of them.

```
C:  minX = smallest of all minX,  maxX = largest of all maxX
A:  convert every box to extents, combine, convert back
```

### Operation 4 — padding

PNG export crops to the selection bounds plus 24px.

```
C:  minX −= 24   minY −= 24   maxX += 24   maxY += 24
A:  x −= 24      y −= 24      w += 48      h += 48
```

Note the `48`. Growing a box by 24 on each side adds 48 to its width, and A is the only form where the number you type isn't the number you meant. This is a small thing that gets typed wrong at three in the morning and produces an export with slightly wrong margins that nobody notices for a month.

### Operation 5 — actually drawing it

The Canvas 2D API takes origin-and-size:

```
A:  ctx.strokeRect(r.x, r.y, r.w, r.h)
C:  ctx.strokeRect(r.minX, r.minY, r.maxX − r.minX, r.maxY − r.minY)
```

A is a direct spread. C converts — at the one call site that runs most often, once per shape per frame.

### Where that leaves it

```
C is more direct for:  building, overlap, union, padding
A is more direct for:  drawing
```

Four to one on operation count. But two things pull the other way, and they aren't nothing:

**A already exists in the schema.** `TextBox` stores `position` plus `width` and derives its height. That is origin-and-size. Choosing C for `Rect` means the board file describes boxes two different ways depending on which object you're reading.

**A is what the ecosystem uses.** Excalidraw and tldraw both store origin-and-size on their elements and derive `minX`/`maxX` through helpers. Two independent canvas engines landing on the same tradeoff is evidence the conversion cost is genuinely small in practice.

The counter-argument to both: **the derivations go behind helper functions either way.** Written once, `maxX(r)` reads the same at every call site regardless of which form is underneath. What survives that flattening is the file format itself — how the board reads on disk, and whether it's internally consistent.

That is the decision, and it is yours to make. Both are defensible; they are not equally defensible for the same reasons.

---

## 4 · Normalization

You drag a marquee from the **bottom-right to the top-left**. Everyone does this; it's not an edge case.

Start `(500, 400)`, end `(100, 100)`. Build a box from that naively:

```
x = start.x = 500
y = start.y = 400
w = end.x − start.x = 100 − 500 = −400
h = end.y − start.y = 100 − 400 = −300
```

A rectangle with **negative width**. It describes a real region — the same region as `{ x: 100, y: 100, w: 400, h: 300 }` — but it describes it backwards.

### What that costs

Take the overlap test from §3, which assumes `x` is the left edge, and feed it this box. Is a point at `x = 300` inside?

```
is 300 >= box.x (500)?          no  → outside
```

`300` is squarely inside that region. The test says no. It doesn't crash, doesn't warn, doesn't log — it confidently returns the wrong answer, and it does so *only* when you happen to have dragged right-to-left. Half your marquees work and half don't, depending on which way your hand moved.

Now multiply. Every place that reads a `Rect` — hit-testing, marquee, connector anchoring, export cropping, the renderer — has the same assumption baked in. You can fix it two ways:

**Defensively, everywhere.** Every consumer checks for negative width and handles it. Forty call sites, each needing the same four lines, each an opportunity to forget. New code written next year has to know to do it.

**Once, at the boundary.** The moment a drag becomes a `Rect`, rewrite it into the one agreed form:

```
if (w < 0) { x = x + w; w = −w }
if (h < 0) { y = y + h; h = −h }

{ x: 500, y: 400, w: −400, h: −300 }  →  { x: 100, y: 100, w: 400, h: 300 }
```

Same region. One representation.

That rewriting is called **normalizing**, and doing it at the boundary turns a thing every consumer must remember into a thing no consumer can encounter. Everything downstream may assume `x` is the left edge, because nothing that isn't has been allowed to become a `Rect`.

### Why it belongs in the spec

An assumption every consumer relies on is called an **invariant**, and an invariant that lives only in the author's head is one refactor away from gone. Writing *"`w` and `h` are never negative"* into the Data Model is what makes it a rule the next person inherits rather than a pattern they might notice.

This is also why the choice in §3 has a second dimension. Form C has this invariant built into its field names — `minX` is called `minX`, so a `minX` larger than its `maxX` looks wrong on sight. Form A has to state it. Neither enforces it; a field is just a number. But one is self-documenting and the other needs the spec to carry the weight.

---

## 5 · Degenerate rectangles

**Degenerate** describes a shape that has collapsed — a rectangle with no width, a triangle whose points are in a line, a line with no length. It's still a valid object. It just has less dimension than its type suggests.

Stem produces them routinely.

**A perfectly horizontal stroke.** Every point has the same `y`. The bounding box is `h = 0` — a rectangle with no height.

**A tap in Draw mode with no movement.** One point. The box is `w = 0, h = 0` — a rectangle that is a point. (What Stem *does* with that stroke is an open tracker item. That it produces a degenerate box is not in question.)

**A `line` shape.** Two of the seven shape kinds are lines, and an axis-aligned one has a flat box by construction.

### What breaks

Not what you'd expect. Degenerate boxes don't crash things — they get quietly discarded by code that was written reasonably.

**The emptiness check.** Somewhere, someone writes the obvious guard:

```
if (r.w > 0 && r.h > 0) { ...test this object... }
```

It reads as "skip empty boxes." It actually means "skip every horizontal line on this board." Your straight lines stop being clickable. The fix is `>=`, and the reason you write `>` in the first place is that a box with no area *feels* like nothing.

**The area ratio.** Stem's ellipse recognition tests the ratio of stroke area to bounding-box area, looking for something near π/4. Draw a flat horizontal stroke and press `Tab`:

```
bounding-box area = 400 × 0 = 0
ratio = stroke area ÷ 0
```

Division by zero. In JavaScript that's `Infinity` or `NaN` rather than a crash, and `NaN` fails every comparison it's in — so the ellipse candidate doesn't rank low, it ranks *incomparably*, and depending on how the sort is written it can land anywhere in the list. `Tab` on a straight line offers you an ellipse. Nothing errored; a sort just got fed a value that isn't a number.

**Intersection with the centre line.** Connector anchoring intersects a box with a line. A box with no height is a segment, and the general intersection routine may return no answer for it. The connector then has nowhere to attach.

### The rule

Degenerate boxes are **legal, common, and never special-cased away.** Comparisons that bound them are inclusive; anything that divides by an extent guards for zero first. Filtering them out at creation looks like defensive hygiene and is actually deleting the user's straight lines.

---

## 6 · What a bounding box throws away

Everything so far has been about what a box gives you. This section is the other half, and it's the one with a live consequence in Stem's spec.

Draw two strokes:

```
        100                    500              100                    500
    100 ┌──────────────────────┐            100 ┌──────────────────────●
        │ ●                    │                │                  ╱   │
        │    ╲                 │                │              ╱       │
        │        ╲             │                │          ╱           │
        │            ╲         │                │      ╱               │
    400 └──────────────────────●            400 ●──────────────────────┘

              stroke A                             stroke B
        drawn top-left ↘ bottom-right         drawn bottom-left ↗ top-right
```

Two visibly different strokes. Now their bounding boxes:

```
A:  left 100, top 100, right 500, bottom 400
B:  left 100, top 100, right 500, bottom 400
```

**Identical.** Not similar — the same four numbers.

That is not a failure. A bounding box is defined to record *extent* — how far the object reaches in each direction — and both strokes reach exactly as far. The box is doing its job perfectly. It simply has no field in which "which diagonal" could be written down, so that information is gone.

A box preserves **where** an object is. It discards **how it was drawn**: the order of the points, the direction of travel, and the path between the extremes.

For most objects that loss is free, because the object's own geometry still has it. A stroke keeps its point array; the box is only the broad phase. The box was never meant to be enough.

### Where it stops being free

`Shape` in the Data Model carries `kind` and `rect`, and nothing else geometric. Stem's recognition snaps a primitive to the source stroke's bounding box.

So press `Tab` on stroke A and cycle to `line`. The shape stores `kind: 'line'` and a box. At render time, something has to draw a line inside that box — and the box offers two diagonals with nothing to choose between them.

**Half the time it renders as the mirror of what you drew.**

`arrow` is worse. It needs the diagonal *and* which end carries the head — two independent binary choices, four combinations, one correct:

```
↘ head at bottom-right     ↘ head at top-left
↗ head at top-right        ↗ head at bottom-left
```

**Three-quarters wrong.**

This is live in three places: `Tab` cycling reaches `line` and `arrow` for every stroke on the board, `Shift`-drawing can produce them, and the insert menu places them directly.

### What it doesn't change

`bounds` still has to be a normalized box. Marquee overlap, export cropping, and union all depend on `minX` genuinely being the smallest — the moment `bounds` starts carrying direction, every operation in §3 breaks. Whatever fixes this cannot fix it there.

Which leaves it as a question about `Shape`: line-like kinds need somewhere to record the information the box cannot hold. Broadly, either the shape stores its two endpoints as an ordered pair — self-describing, and it makes the degenerate-box problem from §5 irrelevant for lines, since you'd no longer be deriving a line from a box at all — or it stores the box plus a couple of flags saying which diagonal and which end.

Both work. Which one Stem takes is a spec decision, and it isn't made here.

---

## Where this connects to the spec

| This document | Spec |
|---|---|
| §1 world/screen | `Spec/Features/Canvas & Viewport.md`, `Spec/Architecture/Rendering.md` |
| §2 bounding boxes | `Spec/Data Model.md` → `BaseObject.bounds` |
| §3 rectangle forms | `Spec/Data Model.md` → `Rect`, `Point` |
| §4 normalization | `Spec/Data Model.md` → `Rect` rules |
| §5 degenerate boxes | `Spec/Features/Drawing.md`, `Spec/Features/Shapes.md` → recognition |
| §6 discarded direction | `Spec/Data Model.md` → `Shape` |
