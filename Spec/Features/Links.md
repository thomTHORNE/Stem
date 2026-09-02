# Links

## What it is

Outbound links to web pages, most often Notion, rendered on the canvas as either a pill or a card. **Outbound only** — nothing about the target is synchronised into the board beyond cached display fields.

Stem is where thinking happens; Notion is where it is written up. A link card is the seam between the two, and it points one way.

See [Data Model — LinkCard](../Data%20Model.md#linkcard) for fields, and [Integrations — Notion](../Integrations/Notion.md) for the fetch and token mechanics.

---

## Behavior

### Two styles

Matching Notion's own vocabulary:

| Style | |
|---|---|
| **Inline mention** | A small pill with the page icon and title, rendered on canvas |
| **Bookmark** | A card with title, description, icon, and domain |

### Inserting

`Cmd+K` opens a prompt, with two input paths:

1. **Paste a URL** — the app fetches the page and reads OpenGraph tags for title, description, and icon. **Works for any URL**, not just Notion.
2. **Search Notion** — queries the Notion API and lists matching pages to pick from. Requires an integration token.

The `Link` entry in the [insert menu](Insert%20Menu.md) is the same action.

### Caching

Title and icon are cached on the `LinkCard` with `fetchedAt`.

**Refresh is on demand**, via a context action — never automatic. A board that opens should not make network requests for every link on it, and a title that changed in Notion is not a reason to redraw a board the user was reading.

### Clicking a link card

In [Select mode](Selection.md), clicking a link card opens the URL in the default browser. Notion desktop intercepts `notion.so` URLs itself, so a Notion link opens in Notion rather than in a browser tab.

### Platform dependency

Fetching requires the Electron main process — a pure browser build has no CORS-free path to an arbitrary URL. The `PlatformAdapter` returns "unsupported" there. See [Architecture](../Architecture/Architecture.md).

This is the one user-visible consequence of the app running in a browser tab before the Electron shell is built, and it is why links are step 10 in the [build order](../Architecture/Build%20Order.md) and the shell is step 11.

---

## States

| State | |
|---|---|
| **Fetched** | `title` and `iconUrl` populated, `fetchedAt` set. |
| **Stale** | `fetchedAt` is old. Stem does not act on this — refresh is on demand only. |

There is no loading state specified for the moment between insert and fetch completion, and no failed-fetch state.

---

## Constraints

- Outbound only. No content is pulled from the target beyond title, description, and icon.
- Refresh is manual. Nothing refreshes on open, on a timer, or on staleness.
- Link insertion is unsupported in the browser build.
- The Notion integration token is never written to a board file — see [Integrations — Notion](../Integrations/Notion.md).

---

## Edge cases

| Scenario | Behavior |
|---|---|
| A URL has no OpenGraph tags | Not yet specified. |
| A fetch fails, or the machine is offline | Not yet specified. |
| `Cmd+K` in the browser build | The `PlatformAdapter` returns "unsupported". What the user sees is not yet specified. |
| A `LinkCard`'s target is deleted or made private | The cached title and icon remain. Refresh behavior against a dead target is not yet specified. |
| Where an inserted link lands — at the cursor or at the viewport center | Unresolved. See TASKS `#8.3`. |
| A bookmark card's `description` | The style renders a description, but `LinkCard` has no `description` field. See TASKS `#1.3`. |
