Status: Open

# Sync

Keeping every board on every device, through a sync service Stem runs.

## Settled

- **Local first.** Every device keeps a full local copy and writes to it first. Sync runs in the background when a connection exists. Offline is not a mode; it is the base everything else rides on.
- **v1 is one user on a free tier.** The architecture assumes many users from the start, so moving to a service for many users is growth, not a rewrite.
- **No encryption on the device in v1.** The service can read synced boards. Informed consent is asked once, at setup.
- **Changes are commands.** The command layer already records every mutation; sync exchanges them per board.
- **Objects merge independently.** The latest write to each object wins, field by field. Deletes are recorded so they do not return.

## Proposed shape

A small API on Cloudflare Workers, board logs, snapshots, and images in R2, and an index in D1. Free-tier figures are from search results and are unverified; the D1 figures conflict.

## Built to grow

v1 serves one user; nothing in it assumes one.

- Every stored record is keyed by user from day one.
- Authentication sits behind one interface. v1 pairs your own devices with a token; accounts replace it later without touching sync.
- The API holds no state between requests, so it scales by adding capacity.
- The sync protocol carries a version number from its first message.
- Storage is laid out per user, then per board. Quotas are configuration, not code.

## Open questions

- Verifying the free-tier limits.
- Compacting a board's change log into a snapshot, and when.
- Uploading images once, and not again on every device.
- The first sync of boards that existed before sync was set up.
- A board edited on two devices, both offline, for days.
- Background sync limits on Android.
- What the consent step shows.

## Findings

- **The command layer is already the change stream.** Sync needs no separate change tracking.

## Dependencies

- **Requires:** Stack — the local store on each platform.
- **Required by:** none.
