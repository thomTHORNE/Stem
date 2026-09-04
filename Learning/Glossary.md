# Glossary

One line per term, plus where it is explained properly. Glance here when a word in a `Learning/` document or in conversation isn't landing.

The glossary defines nothing on its own — each entry is a reminder, and the linked section owns the actual explanation. If the two disagree, the explainer is right.

---

| Term | | In depth |
|---|---|---|
| **AABB** | Axis-aligned bounding box. The form Stem uses everywhere — `bounds` on every object is one. | [Bounds & Rectangles §2](Bounds%20&%20Rectangles.md) |
| **Axis-aligned** | Edges stay parallel to the x and y axes; the box never tilts. Loose, but almost free to test against. | [Bounds & Rectangles §2](Bounds%20&%20Rectangles.md) |
| **Bounding box** | The smallest rectangle that fully contains an object. | [Bounds & Rectangles §2](Bounds%20&%20Rectangles.md) |
| **Broad phase** | A cheap, deliberately imprecise test run over everything, to eliminate most candidates fast. Answers *maybe*. | [Bounds & Rectangles §2](Bounds%20&%20Rectangles.md) |
| **Narrow phase** | The exact, expensive test, run only on what survived the broad phase. Answers *actually*. | [Bounds & Rectangles §2](Bounds%20&%20Rectangles.md) |
| **Cache** | A stored answer that could be recomputed from something else. Faster to read, and wrong the moment its source changes. `bounds` is one. | [Bounds & Rectangles §2](Bounds%20&%20Rectangles.md) |
| **Degenerate** | A shape that has collapsed — a rectangle with no height, a line with no length. Legal and common; every test has to survive it. | [Bounds & Rectangles §5](Bounds%20&%20Rectangles.md) |
| **Extent** | How far an object reaches in a direction. A bounding box records extent and nothing else. | [Bounds & Rectangles §6](Bounds%20&%20Rectangles.md) |
| **Invariant** | Something guaranteed true everywhere, so no consumer has to check it. Only real if it's written down. | [Bounds & Rectangles §4](Bounds%20&%20Rectangles.md) |
| **Normalize** | Rewrite a value into the one agreed form, once, at the boundary — so nothing downstream has to handle variants. | [Bounds & Rectangles §4](Bounds%20&%20Rectangles.md) |
| **Origin** | The corner a rectangle is measured from. In canvas coordinates, top-left; `y` increases downward. | [Bounds & Rectangles §3](Bounds%20&%20Rectangles.md) |
| **RDP** | Ramer–Douglas–Peucker. The simplification pass that reduces a raw ~600-point stroke to roughly 40 on commit. | `Spec/Features/Drawing.md` |
| **Screen space** | Coordinates in pixels on your actual display. Never stored — derived every frame from world space plus the viewport. | [Bounds & Rectangles §1](Bounds%20&%20Rectangles.md) |
| **World space** | Coordinates on the infinite canvas itself, with no viewer and no zoom. What a board file stores. | [Bounds & Rectangles §1](Bounds%20&%20Rectangles.md) |
| **Union** | The single smallest box containing several other boxes. What "fit all content" and whole-board export are built on. | [Bounds & Rectangles §3](Bounds%20&%20Rectangles.md) |
| **Viewport** | `{ x, y, zoom }` — your current view of the plane. A property of the looking, not of the objects. Saved per board. | [Bounds & Rectangles §1](Bounds%20&%20Rectangles.md) |
