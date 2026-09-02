# Command Layer

Every scene mutation is a command object. There is no other way to change the board.

```ts
interface Command {
  do(scene: Scene): void
  undo(scene: Scene): void
  label: string          // for debugging
  coalesceKey?: string   // consecutive commands with the same key merge
}
```

Two stacks, `undo` and `redo`. **Redo clears on any new command.**

---

## Coalescing

`coalesceKey` handles continuous operations. Dragging one object emits many `MoveCommand`s, and they merge into a single undo step keyed on:

```
move:${objectId}:${dragSessionId}
```

The session id is what bounds the merge. Without it, a drag, a pause, and a second drag of the same object would collapse into one undo step — and `Cmd+Z` would undo work the user had already stopped doing.

The same mechanism gives [Erase mode](../Features/Modes.md) its behavior: deleting N objects in one swipe is one undo step, not N.

---

## Build this at step 3

**Before there are many mutation types.** See [Build Order](Build%20Order.md).

Retrofitting undo into thirty scattered call sites is the worst avoidable outcome in this project. Every mutation added after the command layer exists is written through it by default; every mutation added before it has to be found again and rewritten, and the ones that are missed are invisible until a user presses `Cmd+Z` and watches nothing happen.

---

## What this buys elsewhere

The command layer is why several features cost almost nothing to specify:

- **`Tab` tidy is reversible.** Shape recognition replaces a stroke with a primitive as an ordinary command, so `Cmd+Z` restores the original wobble. See [Shapes](../Features/Shapes.md).
- **Silent connector binding is safe.** An endpoint binds with no confirmation and no visual affordance precisely because `Cmd+Z` is the undo path. See [Connectors](../Features/Connectors.md).
- **The crash journal has something to append.** [Persistence](../Features/Persistence.md) journals commands, which is only possible because every mutation is one.

A feature that mutates the scene outside the command layer breaks all three at once.
