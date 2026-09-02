# Board Grid

## What it is

The start screen. A grid of board thumbnails, newest-updated first — the only surface in Stem that is not the canvas.

**Vue-rendered, no canvas.** It reads `index.json` rather than opening board files, which is the reason that index exists. See [Persistence](Persistence.md).

---

## Behavior

### The grid

- Grid of thumbnail cards, ordered by **newest-updated first**.
- Each card shows the thumbnail, the title, and a relative timestamp.

### Actions

| Action | |
|---|---|
| **Open** | Click, or `Enter` |
| **Rename** | `F2`, or double-click the title |
| **Delete** | `Delete`, with confirmation |
| **Duplicate** | |

### Keyboard

- Arrow keys navigate cards.
- `/` focuses a search-by-title field.
- `Cmd+N` creates a board and opens it immediately with an untitled name.

`/` opening a search field here mirrors `/` opening the [insert menu](Insert%20Menu.md) on the canvas: in both places it means "start typing to find something."

### Thumbnails

400×300 PNG, rendered on save. See [Persistence](Persistence.md).

---

## States

| State | |
|---|---|
| **Browsing** | Default. Arrow keys navigate, `Enter` opens. |
| **Searching** | `/` has focused the search field. Typing filters by title. |
| **Renaming** | A card's title is an editable field. |
| **Confirming delete** | A confirmation is pending. |

**Renaming** and **Searching** both capture typing, so the same caution applies here as [the text focus rule](Modes.md#the-text-focus-rule) on the canvas: single-letter shortcuts must be inert while either is active.

The empty state — a first run with no boards — is not yet specified.

---

## Constraints

- Reads `index.json`; does not open board files to render.
- Ordering is newest-updated first and is not user-configurable.
- Search is by title only. Board contents are not searchable in v1.
- No folders, tags, or grouping of boards.
- Delete requires confirmation. It is the only destructive action in Stem that does.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| First run, no boards exist | Not yet specified. |
| A board file exists in `boards/` but not in `index.json` | Not yet specified — see [Persistence](Persistence.md). |
| A thumbnail is missing or fails to render | Not yet specified. |
| A board is deleted | Confirmation is required. Whether the file is removed or moved to trash is not yet specified. |
| A duplicated board's title | Not yet specified. |
| Two boards share a title | Titles are not unique. Search by title returns both. |
