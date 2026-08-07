# Slots and armor

## Changing the item in hand

Switching the slot directly is visible to you and to the server:

```kotlin
slots.selectSilently(Slot.hotbar(3))
slots.sync()                        // tell the server the current slot
```

More often you want something else: grab an item for one hit and put everything back. That is what `using` is for:

```kotlin
val mace = inventory.findInHotbar("mace")
if (mace.found()) {
    slots.using(mace) {
        interaction.attack(target)
    }
}
```

Inside the block the item is already in hand; after the block the slot restores itself — even if something inside threw.

## When you want manual control

`slots.select(...)` returns an object that lets you decide when to go back:

```kotlin
val held = slots.select(Slot.hotbar(5))

held.originalSlot()      // where we were
held.slot()              // where we are

held.restore()           // go back right now
held.restoreWhenSafe()   // go back when it will not hurt (after the swing, say)
held.keep()              // do not go back, stay on the new slot
```

`restoreWhenSafe()` is the one you want in combat: an instant restore can eat the hit.

## Armor

```kotlin
val best = armor.bestSlotFor(ArmorSlot.CHESTPLATE)
if (best.found()) {
    // that slot holds a better chestplate than the one you wear
}

armor.isBetterThanEquipped(ArmorSlot.HELMET, someItem)
```

`bestSlotFor` weighs protection, enchantments and durability, and returns `Slot.NONE` when nothing beats what you wear.

## Putting it on

How you equip depends on where the item is:

```kotlin
fun equip(slot: Slot, armorSlot: ArmorSlot) {
    // in the hotbar — just use it
    if (slot.inHotbar()) {
        slots.using(slot) { interaction.useItem(Hand.MAIN_HAND) }
        return
    }
    // the armor slot is empty — a shift click is enough
    if (inventory.armor(armorSlot).empty()) {
        inventory.shiftClick(slot)
        return
    }
    // otherwise swap through a spare hotbar cell
    val hotbar = Slot.hotbar(inventory.selected().index() % 8 + 1)
    inventory.batch {
        it.swap(slot, hotbar)
        it.delay(1)
        it.swap(Slot.armor(armorSlot), hotbar)
        it.delay(1)
        it.swap(slot, hotbar)
    }
}
```

## Do not rush

Shuffling the inventory every tick is a reliable way to fall out with the server. Put a delay between actions and stay out while the client is busy:

```kotlin
val sinceSwap = timer()

on<PrePlayerTickEvent> {
    if (inventory.busy()) return@on
    if (!sinceSwap.passed(3)) return@on

    val best = armor.bestSlotFor(ArmorSlot.HELMET)
    if (!best.found()) return@on

    equip(best, ArmorSlot.HELMET)
    sinceSwap.reset()
}
```

`timer()` is covered in [Timers and tasks](../extras/tasks.md).
