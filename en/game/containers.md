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

## Recipes

`recipes` is the recipe book: the entries the server sent you. Each one carries a numeric id for this session, and with it the server **places the ingredients into the grid itself**, like clicking a recipe in the green book:

```kotlin
val entry = recipes.find("experience_bottle").firstOrNull()
if (entry != null && container.open()) {
    recipes.craft(entry.id(), true)    // true — craft a whole stack at once
}
```

| | |
|---|---|
| `all()` | every recipe book entry |
| `find(itemId)` | the entries whose result is that item |
| `craft(id[, craftAll])` | ask the server to place the ingredients; `craftAll = true` fills up to a stack |

An entry has `id()`, `resultId()`, `resultCount()`, `category()` and `craftable()` — whether your inventory holds the ingredients right now.

### The grid and the ingredients

What goes where is visible too — `kind()`, `width()`, `height()`, `station()` and `ingredients()`:

```kotlin
val entry = recipes.find("crafting_table").first()
for (row in 0 until entry.height()) {
    val line = (0 until entry.width()).joinToString(" | ") { column ->
        entry.ingredients()[row * entry.width() + column].firstOrNull()?.id() ?: "-"
    }
    chat.print(line)
}
```

`ingredients()` is a list of slots, and each slot is a list of the items it accepts: a recipe with a tag ("any planks") gives you every option, an empty grid slot gives an empty list.

| `kind()` | What is in `ingredients()` |
|---|---|
| `SHAPED` | the grid row by row, `width() * height()` slots, empty ones included |
| `SHAPELESS` | ingredients with no positions, `width()` and `height()` are 0 |
| `FURNACE` | one slot — what gets smelted |
| `STONECUTTER` | one slot — what gets cut |
| `SMITHING` | three slots: template, base, addition |
| `OTHER` | empty |

`station()` is the block it is crafted on: `"minecraft:crafting_table"`, `"minecraft:furnace"`, … An empty string when the server sent no block.

* `craft` clicks the recipe in the handler that is **open right now**: open the crafting table first — or your own inventory for 2×2 recipes. The server puts the output into the result slot; collecting it is on you — `shiftClick`.
* An `id` lives for one session: the server hands the indices out on join. Don't store it between sessions — look the entry up again by `resultId()`.
* The server only places recipes it knows and sent to the book. Custom plugin crafts that are not in the recipe book cannot be crafted this way.

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
| `onFinish { ... }` | runs once the batch has finished; an empty batch drops it too |

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
