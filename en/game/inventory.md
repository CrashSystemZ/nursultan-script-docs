# Inventory and items

`inventory` is `game.inventory()` — the 36 main slots plus the four armor slots and the off-hand. Reads are free; every mutation is queued onto the client's move queue and needs the client thread.

```kotlin
on<PrePlayerTickEvent> {
    val slot = inventory.find("golden_apple")
    if (!slot.found() || inventory.busy()) return@on
    val stack = inventory.item(slot)
    chat.print("${stack.count()} x ${stack.name()}")
    inventory.shiftClick(slot)
}
```

## Addressing a slot

| Method | Type | Description |
|---|---|---|
| `Slot.hotbar(index)` | `Slot` | hotbar slot by index (throws ScriptException outside 0..8) |
| `Slot.inventory(index)` | `Slot` | main-inventory slot, below 9 yields kind HOTBAR (throws ScriptException outside 0..35) |
| `Slot.armor(slot)` | `Slot` | armor slot from an `ArmorSlot` (throws ScriptException on null) |
| `Slot.offhand()` | `Slot` | the off-hand slot |
| `Slot.NONE` | `Slot` | not-found constant, kind NONE and index -1 |
| `Slot.HOTBAR_SIZE` | `int` | constant 9 |
| `Slot.MAIN_SIZE` | `int` | constant 36 |
| `slot.kind()` | `Slot.Kind` | which part of the inventory the slot addresses |
| `slot.index()` | `int` | index inside the kind, -1 for NONE, ArmorSlot ordinal for ARMOR |
| `slot.found()` | `boolean` | kind is not NONE |
| `slot.inHotbar()` | `boolean` | kind is HOTBAR |
| `slot.isArmor()` | `boolean` | kind is ARMOR |
| `slot.isOffhand()` | `boolean` | kind is OFFHAND |
| `slot.hotbarIndex()` | `int` | hotbar index 0..8, -1 when not a hotbar slot |
| `slot.armorSlot()` | `ArmorSlot?` | the armor slot, null when kind is not ARMOR |
| `slot.toString()` | `String` | `none`, `hotbar N`, `inventory N`, `armor HELMET` or `offhand` |

### Slot.Kind

| Constant | Description |
|---|---|
| `NONE` | not found or absent slot |
| `HOTBAR` | hotbar, index 0..8 |
| `INVENTORY` | main inventory, index 9..35 |
| `ARMOR` | armor slot, index is the `ArmorSlot` ordinal |
| `OFFHAND` | off-hand slot, index 0 |

### ArmorSlot

| Constant | Description |
|---|---|
| `HELMET` | head slot, ordinal 0 |
| `CHESTPLATE` | chest slot, ordinal 1 |
| `LEGGINGS` | legs slot, ordinal 2 |
| `BOOTS` | feet slot, ordinal 3 |

## Reading

| Method | Type | Description |
|---|---|---|
| `inventory.held()` | `Item` | main-hand stack, empty item when nothing held |
| `inventory.offhand()` | `Item` | off-hand stack |
| `inventory.item(slot)` | `Item` | stack in that slot (throws ScriptStateException on a not-found slot) |
| `inventory.armor(slot)` | `Item` | equipped stack in that armor slot (throws ScriptException on null) |
| `inventory.items()` | `List<Item>` | immutable list of the 36 main stacks in index order |
| `inventory.selected()` | `Slot` | currently selected hotbar slot |
| `inventory.count(itemId)` | `int` | total count of that item across the 36 main slots |
| `inventory.full()` | `boolean` | no empty stack among the 36 main slots |
| `inventory.empty()` | `boolean` | all 36 main slots are empty |
| `inventory.wearing(itemId)` | `boolean` | that item is equipped in one of the four armor slots |

Item ids default to the `minecraft:` namespace, so `"mace"` and `"minecraft:mace"` are the same id.
No read returns null: an absent stack comes back as an item whose `empty()` is true.

## Searching

| Method | Type | Description |
|---|---|---|
| `inventory.find(itemId)` | `Slot` | first main-inventory slot holding that item, `Slot.NONE` when absent |
| `inventory.find(filter)` | `Slot` | first main-inventory slot passing the filter (throws ScriptException on a null filter) |
| `inventory.findByTag(tagId)` | `Slot` | first main-inventory slot in that item tag, leading `#` stripped |
| `inventory.findInHotbar(itemId)` | `Slot` | first hotbar slot 0..8 holding that item |
| `inventory.findInHotbar(filter)` | `Slot` | first hotbar slot 0..8 passing the filter |
| `inventory.findUsable(filter)` | `Slot` | first matching slot that is non-empty and not on cooldown |
| `inventory.findAll(filter)` | `List<Slot>` | every matching main-inventory slot in index order |

## Moving things

| Method | Type | Description |
|---|---|---|
| `inventory.click(slot, rightButton)` | `void` | queues a PICKUP click, button 1 when rightButton (main thread only) |
| `inventory.shiftClick(slot)` | `void` | queues a QUICK_MOVE click (main thread only) |
| `inventory.swap(slot, hotbarSlot)` | `void` | queues a SWAP into that hotbar slot (main thread only) (throws ScriptException when hotbarSlot is not a hotbar slot) |
| `inventory.drop(slot, wholeStack)` | `void` | queues a THROW, button 1 for the whole stack (main thread only) |
| `inventory.dropHeld(wholeStack)` | `void` | drops the selected hotbar stack directly (main thread only) |
| `inventory.batch(actions)` | `void` | enqueues the collected actions as one move sequence (main thread only) |
| `inventory.busy()` | `boolean` | the client inventory-move queue is non-empty |

### Inventory.Batch

| Method | Type | Description |
|---|---|---|
| `batch.click(slot, rightButton)` | `Batch` | appends a PICKUP click |
| `batch.shiftClick(slot)` | `Batch` | appends a QUICK_MOVE click |
| `batch.swap(slot, hotbarSlot)` | `Batch` | appends a SWAP (throws ScriptException when hotbarSlot is not a hotbar slot) |
| `batch.drop(slot, wholeStack)` | `Batch` | appends a THROW, button 1 for the whole stack |
| `batch.delay(ticks)` | `Batch` | appends a pause in game ticks (throws ScriptException when ticks is 0 or less) |
| `batch.onFinish(action)` | `Batch` | runs the action when the queue closes (API 2) (throws ScriptException when action is null) |

Every mutation needs an open screen handler, and with a non-player screen open the `ARMOR` and `OFFHAND` slots throw `ScriptStateException`; a batch does not block, its queue drains over the following ticks.
Slots of an open chest or server menu are addressed by [Containers](containers.md); switching the held hotbar slot is [Slots and armor](../actions/slots.md).

## The stack

| Method | Type | Description |
|---|---|---|
| `item.empty()` | `boolean` | the stack is empty |
| `item.id()` | `String` | namespaced item id, `minecraft:air` for an empty stack |
| `item.name()` | `String` | plain display name |
| `item.count()` | `int` | stack size |
| `item.maxCount()` | `int` | maximum stack size |
| `item.damage()` | `int` | current damage value |
| `item.maxDamage()` | `int` | maximum durability, 0 when not damageable |
| `item.damageable()` | `boolean` | stack can take durability damage |
| `item.stackable()` | `boolean` | stack can hold more than one item |
| `item.unbreakable()` | `boolean` | has the UNBREAKABLE component |
| `item.enchanted()` | `boolean` | has at least one enchantment |
| `item.rarity()` | `Rarity` | vanilla stack rarity |
| `item.isA(itemId)` | `boolean` | id matches, false for an empty stack |
| `item.hasTag(tagId)` | `boolean` | item is in that tag, leading `#` stripped |

## What it can do

| Method | Type | Description |
|---|---|---|
| `item.placeable()` | `boolean` | BlockItem whose default state has a collision shape |
| `item.buildable()` | `boolean` | BlockItem: full cube, no block entity, default friction/multipliers, not falling or magma (API 2) |
| `item.food()` | `boolean` | has the FOOD component |
| `item.nutrition()` | `int` | food nutrition points, 0 when not food |
| `item.saturation()` | `float` | food saturation value, 0 when not food |
| `item.useAction()` | `String` | lowercase use action: `none`, `eat`, `bow`, `block` |
| `item.disablesBlocking()` | `boolean` | WEAPON component disables shield blocking |
| `item.enchantments()` | `Map<String, Integer>` | namespaced enchantment id to level, insertion-ordered |
| `item.enchantmentLevel(id)` | `int` | level for that id, 0 when absent, `minecraft:` added when unqualified (throws `ScriptException` when the id is blank) |
| `item.attributeModifiers()` | `List<AttributeModifier>` | ATTRIBUTE_MODIFIERS entries, empty when absent |
| `item.containerItems()` | `List<Item>` | CONTAINER contents, else BUNDLE_CONTENTS, else empty |

### AttributeModifier

| Method | Type | Description |
|---|---|---|
| `modifier.attribute()` | `String` | namespaced attribute id the modifier applies to |
| `modifier.id()` | `String` | modifier identifier |
| `modifier.value()` | `double` | modifier amount |
| `modifier.operation()` | `String` | `add_value`, `add_multiplied_base` or `add_multiplied_total` |
| `modifier.slot()` | `String` | equipment-slot group: `any`, `mainhand`, `head` |

## Cooldowns

| Method | Type | Description |
|---|---|---|
| `item.onCooldown()` | `boolean` | this stack's cooldown group is cooling down, false out of world |
| `item.cooldownProgress()` | `float` | remaining cooldown fraction at tickDelta 0, counts down 1..0 (API 2) |
| `item.cooldownProgress(tickDelta)` | `float` | the same fraction interpolated by tickDelta, 1..0 (API 2) |
| `item.setCooldown(ticks)` | `void` | client-side cooldown of that many ticks, 0 or less removes it (API 2) (main thread only) (throws ScriptStateException when out of world) |
| `item.removeCooldown()` | `void` | clears this group's client-side cooldown (API 2) (main thread only) (throws ScriptStateException when out of world) |

A cooldown belongs to the item's cooldown group, not to the stack, so every stack of that item reports the same value.
`setCooldown` and `removeCooldown` write only the client's copy; the server keeps its own, and the next cooldown packet for that group overwrites yours.

## Names, lore, components

| Method | Type | Description |
|---|---|---|
| `item.displayName()` | `Text` | styled display name |
| `item.customName()` | `String?` | plain CUSTOM_NAME text, null when absent |
| `item.lore()` | `List<Text>` | styled LORE lines, empty when absent |
| `item.tooltip(advanced)` | `List<Text>` | vanilla tooltip lines, F3+H version when advanced is true |
| `item.tags()` | `List<String>` | namespaced item tag ids on this item |
| `item.hasComponent(id)` | `boolean` | data component present, false for an unknown id |
| `item.component(id)` | `String?` | `toString()` of the component value, null when absent |
| `item.nbt()` | `String` | CUSTOM_DATA compound as text, empty string when absent |
| `item.customData(key)` | `String?` | string value of that key in CUSTOM_DATA, null when missing |

`Text` values are [styled text](../ui/text.md).

## Rarity

| Constant | Description |
|---|---|
| `COMMON` | white name |
| `UNCOMMON` | yellow name |
| `RARE` | aqua name |
| `EPIC` | light purple name |

## Hands

### Hand

| Constant | Description |
|---|---|
| `MAIN_HAND` | main hand |
| `OFF_HAND` | off hand |

### Arm

| Constant | Description |
|---|---|
| `LEFT` | left arm |
| `RIGHT` | right arm |

`Arm` is `nursultan.item.Arm`, the main-arm side reported by `RenderItemEvent.arm()` and the client-settings packets.
