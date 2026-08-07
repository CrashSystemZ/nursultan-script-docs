# Containers

`container` is the screen the server has open in front of you — a chest, a shulker, a villager, a shop menu. It lets you read what is in it and click it.

```kotlin
on<ClientTickEvent> {
    val screen = game.screen() ?: return@on
    if (screen.title().string() != "Sell") return@on

    val slot = container.find("minecraft:cobblestone")
    if (slot.found()) {
        container.shiftClick(slot)
    }
}
```

## Slot numbering is not the same as the inventory

[`inventory`](inventory.md) addresses **your own** slots — `Slot.hotbar(3)`, `Slot.armor(...)`. `container` addresses the **open handler**, which is a flat list of slots the server laid out, and it uses plain indices wrapped in a `ContainerSlot`.

The handler always puts the container's own slots first and your inventory after them:

| Range | What it is |
|---|---|
| `0` … `storage() - 1` | the container — 54 slots for a six-row chest |
| `storage()` … `storage() + 26` | your main inventory rows |
| `storage() + 27` … `size() - 1` | your hotbar |

```kotlin
container.syncId()      // the id the server gave this menu
container.size()        // every slot, container and your inventory together
container.storage()     // how many leading slots belong to the container
container.open()        // false when the only thing open is your own inventory
```

With no container open, this is your own inventory handler and `storage()` is the crafting result and grid.

`playerSlot(...)` converts back, so you can hand a container slot to the rest of the API:

```kotlin
val slot = container.find("minecraft:totem_of_undying")
val mine = container.playerSlot(slot)      // Slot.NONE if it is in the container itself
if (mine.found()) {
    slots.using(mine) { interaction.useItem(Hand.MAIN_HAND) }
}
```

## Reading

```kotlin
container.item(0)                          // by raw index
container.item(slot)                       // by ContainerSlot
container.items()                          // everything, in handler order

container.find("minecraft:diamond")        // first slot holding it
container.find("diamond", 0, container.storage())   // only inside the container
container.find { it.enchantmentLevel("sharpness") >= 4 }
container.findAfter(10) { !it.empty() }
container.findAll { it.hasTag("swords") }

container.count("minecraft:emerald")
container.empty()                          // first free slot, or NONE
container.empty(0, container.storage())
```

`item(...)` never returns `null` — an empty slot gives you an empty item, so check `empty()`.

## Clicking

```kotlin
container.click(slot)                          // left click
container.click(slot, 1)                       // right click
container.click(slot, 0, SlotAction.QUICK_MOVE)
container.shiftClick(slot)                     // the same, shorter
container.clickButton(0)                       // a screen button: stonecutter, loom, villager trade
```

| Action | What it does |
|---|---|
| `PICKUP` | pick the stack up / put it down — the default |
| `QUICK_MOVE` | shift-click: moves a whole stack to the other side in one packet |
| `SWAP` | swap with a hotbar slot, `button` is `0..8` |
| `THROW` | throw out of the window |
| `PICKUP_ALL` | double-click, gathers matching stacks |
| `QUICK_CRAFT` | the drag-across-slots gesture |
| `CLONE` | middle click, creative only — it throws in survival |

`QUICK_MOVE` is what you want for moving things: one click moves the whole stack, instead of picking it up and putting it down twice.

## Sequences and delays

A single click goes out immediately. When you need several in order — and the server needs a tick between them to keep up — queue them as one batch:

```kotlin
container.batch {
    it.click(from, 0, SlotAction.SWAP)
    it.delay(1)
    it.click(to, 0, SlotAction.SWAP)
    it.delay(1)
    it.click(from, 0, SlotAction.SWAP)
}
```

The whole batch is one queue: the client sends the first click, waits the delay in **game ticks**, sends the next. Nothing blocks — your handler returns straight away and the queue drains on its own over the following ticks.

| | |
|---|---|
| `click(slot[, button[, action]])` | as above |
| `shiftClick(slot)` | |
| `swap(slot, hotbarIndex)` | `hotbarIndex` is `0..8` |
| `drop(slot, wholeStack)` | |
| `delay(ticks)` | wait before the next step, 1 tick or more |

`container.busy()` is `true` while a queue is still draining — yours or the client's own. Wait it out before starting another:

```kotlin
on<ClientTickEvent> {
    if (container.busy()) return@on
    val slot = container.find { !it.empty() }
    if (slot.found()) {
        container.shiftClick(slot)
    }
}
```

The client does not slow your clicks down for you. How fast is safe depends on the server, so pace them yourself with `delay` and `busy()` — an instant chest stealer is the fastest way to get your users banned.

## Things to keep in mind

* **The screen can close between reading and clicking.** The slot you found a tick ago may not exist any more; the click is validated against the handler that is open right now and throws if the index is out of range.
* Clicks only work **on the client thread**. From a packet handler, wrap them in `onClientThread { }`.
* Armor and offhand are not part of a foreign handler. `inventory.click(Slot.armor(...))` with a chest open says so instead of clicking something random.
* To close the screen use `game.closeScreen()`.
* Servers reorder their menus between ticks. Match on the item, not on the index you saw last time.
