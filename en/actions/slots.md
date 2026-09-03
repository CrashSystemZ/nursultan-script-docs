# Slots and armor

`slots` is `client.slots()` and `armor` is `client.armor()`. `select` returns a handle that remembers the slot selected before it; `armor` compares a candidate piece against what is worn. `Slot` and `ArmorSlot` themselves are documented on [Inventory and items](../game/inventory.md).

```kotlin
on<ClientTickEvent> {
    slots.using(Slot.hotbar(2)) {
        interaction.useItem(Hand.MAIN_HAND)
    }

    val best = armor.bestSlotFor(ArmorSlot.HELMET)
    if (best.found()) inventory.shiftClick(best)
}
```

## Selecting a slot

| Method | Type | Description |
|---|---|---|
| `slots.select(hotbarSlot)` | `HeldSlot` | selects the hotbar slot, syncs it, returns a restore handle (main thread only) (throws `ScriptException` when the slot is not a hotbar slot) |
| `slots.selectSilently(hotbarSlot)` | `void` | selects and syncs the slot, records nothing to restore (main thread only) (throws `ScriptException` when the slot is not a hotbar slot) |
| `slots.sync()` | `void` | resends the currently selected hotbar slot to the server (main thread only) |
| `slots.hold(hotbarSlot)` | `HeldSlot` | reuses the live handle when that slot is already selected, selects it otherwise (main thread only) |
| `slots.selected()` | `Slot` | hotbar slot selected right now |
| `slots.using(slot) { }` | `T` | selects, runs the block, calls `restoreWhenSafe()` in a `finally` (main thread only) |

All of them throw `ScriptStateException` when there is no world and after the script is unloaded. Every live handle of a script is restored silently when the script unloads.
A deferred restore happens on a tick of the client's own, so a handle is not proof of what the hand holds: `hold` and `holding()` compare against the slot that is really selected.

## The handle

| Method | Type | Description |
|---|---|---|
| `originalSlot()` | `int` | hotbar index 0..8 selected when `select` ran |
| `slot()` | `int` | hotbar index 0..8 this handle selected |
| `holding()` | `boolean` | that slot is still the selected one and the handle is not spent |
| `restore()` | `void` | switches back now and syncs it |
| `restoreWhenSafe()` | `void` | switches back on a later tick with no attack |
| `keep()` | `void` | drops the pending restore, the slot stays selected |
| `close()` | `void` | same as `restoreWhenSafe()` |

`restore`, `restoreWhenSafe`, `keep` and `close` are mutually exclusive: the first call takes effect and the rest are no-ops. They switch to the client's global last-selected slot, which a manual hotbar change overwrites, so it is not guaranteed to equal `originalSlot()`.

## Armor

| Method | Type | Description |
|---|---|---|
| `armor.bestSlotFor(slot)` | `Slot` | best main-inventory slot 0..35 for that armor slot, `Slot.NONE` when nothing beats the worn piece |
| `armor.isBetterThanEquipped(slot, candidate)` | `boolean` | true when the **equipped** piece wins or ties (`>=`) — inverted relative to the name (throws `ScriptException` when the item is not from the inventory) |

`bestSlotFor` ranks by protection (armor + toughness + protection enchants), then remaining durability, then attribute modifier, then Unbreaking level. Both methods throw `NullPointerException` outside a world and have no thread guard.
