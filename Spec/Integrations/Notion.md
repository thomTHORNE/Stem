# Notion

Notion is Stem's only integration. It is **outbound only**: Stem links to Notion pages and caches enough to render a card. Nothing is written to Notion, and nothing is synchronised back.

The relationship reflects how the two tools are used together — Stem is where thinking happens, Notion is where it is written up. See [Links](../Features/Links.md) for the behavior; this document holds the mechanics.

---

## Two fetch paths

`Cmd+K` opens a prompt with two input paths. Only the second is Notion-specific.

### Paste a URL

The app fetches the page and reads **OpenGraph tags** for title, description, and icon.

This path **works for any URL**, not just Notion. It requires no token and no configuration.

### Search Notion

Queries the Notion API `/v1/search` endpoint and lists matching pages to pick from.

**Requires an integration token**, and requires pages to be shared with the integration — a Notion integration sees nothing by default.

---

## Why this works in Electron

Running inside Electron means **no CORS restriction on either fetch** — the main process makes the request, and the main process is not a browser context.

A pure browser build would need a proxy, which means a server, which Stem does not have. The `PlatformAdapter` returns "unsupported" in the browser build rather than degrading. See [Architecture](../Architecture/Architecture.md).

This is the one capability that is genuinely absent before the Electron shell exists, and it is why links are step 10 and the shell is step 11 in the [build order](../Architecture/Build%20Order.md) — links are the last thing built that can run in a browser tab, and the first that would want the shell.

---

## Token storage

The Notion integration token is stored in Electron **`safeStorage`** — Keychain on macOS, DPAPI on Windows. It is configured in [Settings](../Features/Settings.md).

**Never in the board file.** A board is a small readable file the user may put in iCloud or share; a credential does not belong in one.

---

## Caching

Title and icon are cached on the [`LinkCard`](../Data%20Model.md#linkcard) with `fetchedAt`.

**Refresh is on demand only**, via a context action. Nothing refreshes on board open, on a timer, or on staleness. Opening a board makes no network requests.

---

## Opening a link

Clicking a link card in Select mode opens the URL in the default browser. **Notion desktop intercepts `notion.so` URLs itself**, so a Notion link opens in the Notion app rather than in a browser tab. Stem does nothing special to arrange this.

---

## Constraints

- Outbound only. Stem never writes to Notion.
- The search path requires an integration token and pages explicitly shared with the integration.
- Both fetch paths require the Electron shell. Neither works in the browser build.
- The token is never written to a board file.

---

## Edge cases

| Scenario | Behavior |
|---|---|
| No integration token is configured | The paste-a-URL path still works; search does not. What the user sees is not yet specified. |
| A page is not shared with the integration | It does not appear in search results. Notion returns nothing rather than an error. |
| A token is revoked in Notion | Not yet specified. |
| A URL has no OpenGraph tags | Not yet specified — see [Links](../Features/Links.md). |
| A fetch fails or the machine is offline | Not yet specified. |
