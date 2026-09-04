# Persistence

## What it is

How a board gets to disk and back. **One JSON file per board** — no binary format, no database. A board is a small readable file, and that is a deliberate property rather than an implementation shortcut.

Autosave is continuous and silent. There is no save action, no dirty indicator, and no save dialog.

See [Data Model — BoardFile](../Data%20Model.md#boardfile) for the file's schema and [Board Grid](Board%20Grid.md) for the screen that lists boards.

---

## Behavior

### Storage layout

| Path | Contents |
|---|---|
| `<userData>/boards/<id>.json` | One board |
| `<userData>/index.json` | `{ id, title, createdAt, updatedAt, thumbnailPath }` per board, for fast grid rendering |
| `<userData>/thumbnails/` | 400×300 PNG per board, rendered on save |

The index exists so the [board grid](Board%20Grid.md) can render without opening every board file. It is derived data — the board files are authoritative.

The boards directory is **user-configurable** in [Settings](Settings.md), so it can be pointed at iCloud or Dropbox.

### Autosave

- Debounced save **800ms** after the last mutation.
- Plus a hard save every **30s** during continuous activity.
- Save on window blur and on `before-quit`.

The debounce handles the common case of a pause between marks. The 30-second hard save handles the case the debounce cannot: continuous drawing that never leaves an 800ms gap, where a debounce alone would never fire.

### Atomic writes

Write to `<id>.json.tmp`, `fsync`, then rename. **Never truncate the live file.**

A rename is atomic on both target platforms, so a crash during a save leaves either the previous board or the new one — never a half-written file. Truncating in place makes losing a board a matter of timing.

### Crash journal

A crash journal appends the last N commands to `<id>.journal`. On open, if a journal is newer than the board file, recovery is offered.

Journaling commands rather than snapshots is only possible because every mutation is a command — see [Command Layer](../Architecture/Command%20Layer.md). The value of N is not yet specified.

### Viewport

`viewport` is saved with the board and restored on open. See [Canvas & Viewport](Canvas%20&%20Viewport.md).

A viewport change never marks the board dirty and never schedules a write of its own. Panning is not a document change — it emits no command and cannot be undone, see [Command Layer](../Architecture/Command%20Layer.md) — so it does not earn a save. The viewport rides along with the next write that happens for another reason, and the blur and `before-quit` saves fire regardless of dirty state, so a session spent reading rather than editing still persists where you were looking.

A crash therefore restores the last written viewport, which may be older than the one at the crash. That is accepted: the journal replays what you drew, not where you were standing, and losing the latter costs nothing you made.

---

## States

| State | |
|---|---|
| **Clean** | Board file matches the scene. No pending write. |
| **Dirty** | Scene mutations since the last save. A debounced write is scheduled. |
| **Writing** | A `.tmp` file exists and has not yet been renamed. |
| **Journal ahead** | On open, `<id>.journal` is newer than `<id>.json`. Recovery is offered. |

None of these is surfaced to the user. There is no dirty indicator and no save spinner — the point of continuous autosave is that saving is not a thing the user thinks about.

---

## Constraints

- One file per board. No partial or incremental writes — the whole `objects` array is written each time.
- Writes are atomic by rename. The live file is never truncated.
- The index and thumbnails are derived. A board file is the authority for its own contents.
- No version history and no undo across sessions. The undo stack is runtime state and does not survive closing a board.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| A crash during a save | The rename either happened or it did not. The previous board file survives intact. |
| A crash between saves | The journal covers the gap, if the last N commands are enough to cover it. Recovery is offered on open. |
| A crash after panning, with no scene mutation since the last save | The board is not dirty and the journal is not ahead, so no recovery is offered. The viewport is restored as of the last write. Nothing was lost. |
| `index.json` disagrees with the contents of `boards/` | Not yet specified. |
| The boards directory is pointed at iCloud and two machines write the same board | Not yet specified. |
| The boards directory is changed in Settings while boards exist | Not yet specified. |
| A board file fails to parse, or carries an unknown `version` | Not yet specified. |
| Recovery is declined | Not yet specified. |
