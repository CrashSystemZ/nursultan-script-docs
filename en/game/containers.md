# Containers

`container` is `game.container()` — the open screen handler as one flat slot list. Indices `0..storage()-1` are the container's own slots, then 27 inventory slots, then the 9 hotbar slots; every member throws `ScriptStateException` when there is no world or no screen handler.

```kotlin
on<ClientTickEvent> {
    if (!container.open() || container.busy()) return@on
    val slot = container.find("minecraft:diamond", 0, container.storage())
    if (!slot.found()) return@on
    container.batch {
        it.shiftClick(slot)      // one step per tick
        it.delay(2)
    }
}
```

## Slot numbering

| Method | Type | Description |
|---|---|---|
| `container.syncId()` | `int` | sync id of the open screen handler |
| `container.size()` | `int` | total slot count of the open handler |
| `container.storage()` | `int` | `size() - 36`, floored at 0 — the container's own slots |
| `container.open()` | `boolean` | true when the sync id is non-zero, i.e. not the plain player inventory |
| `container.playerSlot(slot)` | `Slot` | handler index as an inventory slot; `Slot.NONE` outside the player section |

| Method | Type | Description |
|---|---|---|
| `ContainerSlot.NONE` | `ContainerSlot` | constant with index -1, meaning not found |
| `ContainerSlot.of(index)` | `ContainerSlot` | wraps a raw handler index (throws `ScriptException` when index < 0) |
| `slot.index()` | `int` | raw handler slot index, -1 for `NONE` |
| `slot.found()` | `boolean` | index is 0 or more |
| `slot.equals(other)` | `boolean` | true when the other `ContainerSlot` has the same index |
| `slot.hashCode()` | `int` | the index itself |
| `slot.toString()` | `String` | `none` or `container <index>` |

`playerSlot` maps the first 27 slots of the player section to `Slot.inventory(9..35)` and the last 9 to `Slot.hotbar(0..8)`.
`Slot` and `Item` are documented on [Inventory and items](inventory.md).

## Reading

| Method | Type | Description |
|---|---|---|
| `container.item(slot)` | `Item` | stack at that `ContainerSlot`; empty item for null or out of range |
| `container.item(index)` | `Item` | stack at that raw index; empty item when out of range |
| `container.items()` | `List<Item>` | every slot in handler order |
| `container.find(itemId)` | `ContainerSlot` | first slot holding that item id; `NONE` when absent |
| `container.find(itemId, from, to)` | `ContainerSlot` | same, restricted to an index range |
| `container.find(filter)` | `ContainerSlot` | first slot passing the filter; `NONE` for a null filter |
| `container.find(from, to, filter)` | `ContainerSlot` | first passing slot inside the range |
| `container.findAfter(from, filter)` | `ContainerSlot` | first passing slot from that index to the end |
| `container.findAll(filter)` | `List<ContainerSlot>` | every passing slot; empty for a null filter |
| `container.count(itemId)` | `int` | summed stack counts of that item over the handler |
| `container.count(itemId, from, to)` | `int` | same, over an index range |
| `container.empty()` | `ContainerSlot` | first empty slot, or `NONE` |
| `container.empty(from, to)` | `ContainerSlot` | first empty slot inside a range |

Index ranges are half-open and clamped to `0..size()`.
Item ids default to the `minecraft:` namespace; a blank id throws `ScriptException`.

## Clicking

| Method | Type | Description |
|---|---|---|
| `container.click(slot)` | `void` | left click, button 0, `PICKUP` (main thread only) (throws `ScriptException` when the slot is `NONE` or beyond the handler) |
| `container.click(slot, button)` | `void` | click with an explicit mouse button, `PICKUP` (main thread only) |
| `container.click(slot, button, action)` | `void` | click with an explicit slot action (main thread only) |
| `container.shiftClick(slot)` | `void` | click with button 0 and `QUICK_MOVE` (main thread only) |
| `container.clickButton(button)` | `void` | screen-handler button: stonecutter, loom, villager trade (main thread only) |
| `container.busy()` | `boolean` | true while the client's inventory move queue still has actions |

| Constant | Description |
|---|---|
| `PICKUP` | normal left/right click; also the fallback when the action is null |
| `QUICK_MOVE` | shift-click move |
| `SWAP` | hotbar swap, button is the hotbar index 0..8 |
| `CLONE` | middle-click clone (throws `ScriptException` outside creative) |
| `THROW` | drop; button 1 drops the whole stack, 0 drops one item |
| `QUICK_CRAFT` | drag-craft / drag-distribute |
| `PICKUP_ALL` | double-click collect-all |

`busy()` also covers the client's own inventory moves, not only a script's.

## Sequences

| Method | Type | Description |
|---|---|---|
| `container.batch(actions)` | `void` | queues the steps onto the inventory move queue, drained over following ticks (main thread only) (throws `ScriptException` when actions is null) |

| Method | Type | Description |
|---|---|---|
| `it.click(slot)` | `Batch` | queues button 0 / `PICKUP` |
| `it.click(slot, button)` | `Batch` | queues a click with that mouse button / `PICKUP` |
| `it.click(slot, button, action)` | `Batch` | queues a click with an explicit slot action |
| `it.shiftClick(slot)` | `Batch` | queues button 0 / `QUICK_MOVE` |
| `it.swap(slot, hotbarIndex)` | `Batch` | queues `SWAP` onto a hotbar slot (throws `ScriptException` when hotbarIndex is outside 0..8) |
| `it.drop(slot, wholeStack)` | `Batch` | queues `THROW`; button 1 for the whole stack, 0 for one item |
| `it.delay(ticks)` | `Batch` | queues a wait in game ticks (throws `ScriptException` when ticks <= 0) |
| `it.onFinish(action)` | `Batch` | runs on the queue's close hook; the vanilla processor runs it immediately (API 2) (throws `ScriptException` when action is null) |

A batch that queued no click or delay is discarded, and its `onFinish` callbacks never run.

## Recipes

| Method | Type | Description |
|---|---|---|
| `recipes.all()` | `List<RecipeEntry>` | every recipe-book entry in book order (API 2) (main thread only) |
| `recipes.find(resultItemId)` | `List<RecipeEntry>` | entries whose result is that item id (API 2) (main thread only) |
| `recipes.craft(recipeId)` | `void` | clicks the recipe in the open crafting screen, craftAll false (API 2) (main thread only) |
| `recipes.craft(recipeId, craftAll)` | `void` | server fills the open crafting grid; craftAll makes a full stack (API 2) (main thread only) |

## A recipe entry

| Method | Type | Description |
|---|---|---|
| `entry.id()` | `int` | network recipe id, valid only for the current connection (API 2) |
| `entry.resultId()` | `String` | registry id of the displayed result item (API 2) |
| `entry.resultCount()` | `int` | stack size of the displayed result (API 2) |
| `entry.category()` | `String` | recipe-book category registry id; `""` when unknown (API 2) |
| `entry.craftable()` | `boolean` | the inventory currently holds the ingredients (API 2) |
| `entry.kind()` | `RecipeKind` | which display shape `ingredients()` follows (API 2) |
| `entry.width()` | `int` | grid width; 0 for non-`SHAPED` kinds (API 2) |
| `entry.height()` | `int` | grid height; 0 for non-`SHAPED` kinds (API 2) |
| `entry.station()` | `String` | registry id of the crafting station item; `""` when none was sent (API 2) |
| `entry.ingredients()` | `List<List<Item>>` | per slot, the accepted items; empty inner list for an empty cell (API 2) |

| Constant | Description |
|---|---|
| `SHAPED` | crafting grid row by row, `width() * height()` cells, empty ones included (API 2) |
| `SHAPELESS` | unordered crafting ingredients, width and height 0 (API 2) |
| `FURNACE` | one ingredient slot, the smelting input (API 2) |
| `STONECUTTER` | one ingredient slot, the cutting input (API 2) |
| `SMITHING` | three ingredient slots: template, base, addition (API 2) |
| `OTHER` | unrecognised display; `ingredients()` is empty (API 2) |
