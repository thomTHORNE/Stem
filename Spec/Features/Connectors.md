# Connectors

## What it is

Two paths that both produce a connection between objects. They differ in what the connection *looks like*, and that difference is the point — one is for thinking, the other is for showing.

| | Path A (drawn) | Path B (generated) |
|---|---|---|
| Invoked by | drawing a line | `C` + two hint letters |
| Appearance | hand-drawn, keeps your wobble | clean curve, rounded corners |
| On move | rigid warp | reroute |
| Best for | thinking, exploring | settled diagrams, presentable output |

See [Data Model](../Data%20Model.md) for [Stroke](../Data%20Model.md#stroke) bindings and the [Connector](../Data%20Model.md#connector) object.

---

## Behavior

### Path A — drawn, freehand

Draw a line in [Draw mode](Drawing.md). If an endpoint finishes within **24px** of an object's bounds, it binds silently.

The radius is **screen space**, so it is constant regardless of zoom — the tolerance matches what the user can see, not what the world coordinates say.

**No handle, no aim, no mode, no visual affordance to hit.** If the binding was unwanted, `Cmd+Z`. Silent binding is only safe because undo is reliable; see [Command Layer](../Architecture/Command%20Layer.md).

#### When a bound object moves — rigid warp

Compute the delta at each bound endpoint and distribute it across the stroke's points, weighted by normalized arc-length distance from that endpoint:

```
weight(i) = 1 - (arcLengthFromBoundEnd(i) / totalArcLength)
point[i] += delta * weight(i)
```

The line stretches and leans but **keeps every wobble you drew**. Roughly 20 lines of code.

Warping the drawn stroke rather than regenerating it is what keeps a hand-drawn diagram hand-drawn after you have rearranged it. A regenerated curve would quietly erase the character of the mark.

#### Warp threshold

If warping would take the stroke past **2× its `originalLength`**, fall back to regenerating a rough curve between the new anchors.

In practice you nudge shapes a few hundred pixels and never hit this. The threshold exists so that a stroke dragged across the board does not become an unrecognisable smear.

#### On deletion of a bound object

The stroke survives with that end unbound. **Never cascade-delete.**

### Path B — generated, clean

`C` → hint letter on the source → hint letter on the target. A [Connector](../Data%20Model.md#connector) is created: a clean curved path with rounded corners between computed edge anchors, with an arrowhead.

**Zero pointer movement.** Two keystrokes connect two objects anywhere on screen, including objects on opposite sides of a large board. On a trackpad this is faster than drawing, and it is the form you want once a diagram has settled.

#### Pointer fallback

In Connect mode, drag from anywhere *inside* the source to anywhere *inside* the target. The whole shape is the hit target; handles are never rendered. See [Design Principles](../Design%20Principles.md) → No aiming.

#### Anchoring

Compute the line between the two objects' centers, intersect with each object's bounds, and attach there. On move, recompute — **connectors always reroute, never warp.**

#### Routing

`curved` by default: a cubic bézier with control points pushed out along the anchor normals.

`orthogonal` is available via `Tab` cycling on a selected connector, with 8px rounded corners.

### Converting between them

`Tab` on a selected bound freehand stroke converts it to a clean `Connector` between the same two objects. `Tab` again reverts.

This makes the whole tidy-up story one gesture: **`Tab` means "make this presentable."** The same key that cleans a box cleans the line between two boxes.

---

## States

A connection's state is which object holds it and whether its ends are bound:

| State | |
|---|---|
| **Unbound stroke** | A drawn line with `binding.from` and `binding.to` both `null`. An ordinary stroke. |
| **Half-bound** | One end bound. Reached by drawing near one object, or by deleting one of two bound targets. |
| **Bound stroke** | Both ends bound. Warps on move. |
| **Connector** | A generated `Connector` object. Reroutes on move. |

`Tab` moves a bound stroke to **Connector** and back. Deleting a target moves a bound stroke to **half-bound**, never to deleted.

---

## Constraints

- The binding radius is 24px in **screen space**, fixed and not configurable.
- Warping past 2× `originalLength` falls back to regeneration.
- Nothing cascade-deletes. A connection always outlives what it points at.
- Connectors are not rendered with handles, in any mode.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| A bound object is deleted | The stroke survives with that end unbound. Never cascade-delete. |
| Both bound objects are deleted | Both ends unbind by the same rule; the stroke survives as an ordinary stroke. |
| A stroke endpoint is within 24px of two overlapping objects | Not yet specified. |
| A stroke that is not line-like ends near an object | `binding` is a field on every `Stroke`, not only line-like ones. Whether binding applies to any stroke or only to those recognised as lines is not yet specified. |
| The same object is chosen as both source and target in Connect mode | Not yet specified. |
| A `Connector` end holds a free `{ point }` rather than an object | The field permits it, but no interaction that creates one is specified. |
| Warping is undone | Warp is a mutation like any other and passes through the command layer. |
