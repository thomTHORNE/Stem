# Platforms

The research behind running Stem on every device a thought might arrive on, with full parity. What v1 targets is [Scope](../../Spec/Scope.md)'s to say.

The topics are independent. Each develops and promotes on its own schedule.

---

## Why together

Parity is one promise made three times: the same rendering on every platform, the same capabilities under every input, and the same boards on every device. Any one failing breaks the promise, and each constrains the others — the stack decides what input events exist, and where local data lives.

---

## Seams

| Finding | Between | Owned by |
|---|---|---|
| The input model can only use what the framework exposes: pointer kinds, gestures, hardware keyboard detection. | Stack ↔ Touch Input | Touch Input |
| Sync needs a local store on every platform, and background sync behaves differently on Android than on desktop. | Stack ↔ Sync | Sync |

---

## What would falsify it

- **The spike fails on text.** If a rich text box cannot be edited reliably on a zoomable canvas, the working stack choice falls and the fallback takes its place.
- **A capability cannot be reached on touch without aiming.** If parity forces small targets onto the phone, either parity or "no aiming" gives.
- **Sync loses a thought.** One lost edit costs more trust than offline editing earns.
