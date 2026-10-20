# Scope

What each version of Stem includes and excludes, and why.

[Foundation](../Foundation.md) and the [design principles](Design%20Principles.md) stay fixed across versions; scope is what changes. Something not yet decided either way is not listed here — it is in [Ideas.md](../Ideas.md) or `Research/`.

A version includes a feature as a commitment to what the product is. How the feature behaves is specified in `Features/` once it is understood, not before.

A version may withhold a pillar of Foundation, but never break one of its refusals. What ships must stay true to Foundation even where it does not yet deliver all of it.

---

## v1

### Included

| | |
|---|---|
| **The board** | Everything specified in `Features/`: canvas and viewport, modes, drawing, shapes, connectors, selection, writing, the insert menu, links, export, persistence, the board grid, and settings. |
| **Ownership** | Original and reference as Foundation defines them, read-only references, and informed consent before anything leaves the device. |
| **Cognition blocks** | Cognition types defined off the board and placed on it as blocks. |
| **The GitHub integration** | A repository on the board as a block: searching and navigating it, commit, pull, and push, and annotating a rendered document. |
| **Board search** | Finding a board by the words in it: its title and the text it holds. Matches words, not meaning. |

### Excluded

| Excluded | |
|---|---|
| Collaboration | Stem is a personal tool. No multi-user model, no presence, no shared state. |
| Mobile | Desktop only. The interaction model is a trackpad and a full keyboard. |
| Layers | `z` is creation order and there is no UI for it. |
| Grouping | Multi-select is transient and is not persisted. |
| Board templates | Every board starts empty. A cognition type structures a block, not a board. |
| Presentation mode | Stem is for thinking, not presenting. Output leaves via [export](Features/Export.md). |
| Image import | No raster content placed on the canvas. |
| AI | Every AI surface — recollection, conversation, and talking a document through in the GitHub block — arrives together in a later release, on one runtime and one consent flow. Foundation's rules for AI hold now and apply from the first surface built. |
| Recollection | Finding a thought by asking for it in your own words depends on AI and arrives with it. Until then, board search matches words. |
| The meta layer | The least developed pillar. Who writes on it is not yet understood well enough to commit to. |
| Notion search | Linking a Notion page works through a pasted URL like any other link. Searching Notion needs a token and pages shared with an integration — setup friction for a path the pasted URL already covers. |
| Integrations other than GitHub | GitHub is v1's only integration. |

Excluding these is what makes the rest of v1 small enough to finish. Anything here worth reconsidering later lives in [Ideas.md](../Ideas.md), not in this table.
