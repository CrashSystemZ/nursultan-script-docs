# Inventory and items

`inventory` is **your own** inventory — hotbar, main rows, armor, offhand. A chest or a server menu is a different thing with different slot numbering; that one lives in [Containers](containers.md).

## Slots

A slot is the address of a cell, not the item in it. You build one like this:

```kotlin
Slot.hotbar(0)                  // first hotbar cell, 0..8
Slot.inventory(20)              // main inventory cell, 0..35
Slot.armor(ArmorSlot.HELMET)
Slot.offhand()
Slot.NONE                       // "nothing found"
```

Everything that searches for an item returns a slot, and it may come back empty:

```kotlin
val slot = inventory.find("minecraft:golden_apple")
if (!slot.found()) return

slot.inHotbar()
slot.hotbarIndex()
slot.isArmor()
slot.armorSlot()
```

## What is in there

```kotlin
inventory.held()                        // in hand
inventory.offhand()
inventory.item(Slot.hotbar(3))
inventory.armor(ArmorSlot.CHESTPLATE)
inventory.items()                       // the whole inventory as a list
inventory.selected()                    // the selected hotbar slot

inventory.count("minecraft:arrow")
inventory.wearing("minecraft:elytra")
inventory.full()
inventory.empty()
```

## Searching

```kotlin
inventory.find("minecraft:mace")                 // anywhere
inventory.findInHotbar("minecraft:mace")         // hotbar only
inventory.findByTag("swords")                    // by item tag
inventory.find { it.enchantmentLevel("sharpness") >= 4 }
inventory.findInHotbar { it.disablesBlocking() }
inventory.findUsable { it.food() }               // usable right now
inventory.findAll { it.damageable() }            // every match
```

You can drop the `minecraft:` prefix — `"mace"` works the same.

## An item

```kotlin
item.empty()                    // empty cell
item.id()                       // "minecraft:diamond_sword"
item.name()                     // its in-game name
item.count()  item.maxCount()
item.isA("diamond_sword")
item.hasTag("swords")

item.damage()  item.maxDamage()
item.damageable()
item.unbreakable()
item.stackable()

item.enchanted()
item.enchantments()             // all of them
item.enchantmentLevel("sharpness")

item.food()
item.nutrition()
item.saturation()

item.placeable()                // a block you can stand on: not a torch, not a sign

item.rarity()
item.onCooldown()               // the item is on cooldown
item.disablesBlocking()         // an axe can break a shield

item.useAction()                // "eat", "drink", "bow", "block", "none"...
item.tags()                     // every item tag it is in
```

## What the server wrote on it

Servers put almost everything into the name and the lore — price, seller, cooldown, which kit this is. Both come back as [styled text](../ui/text.md):

```kotlin
item.displayName()              // Text: the name as the tooltip draws it
item.customName()               // String: the anvil name, or null
item.lore()                     // List<Text>: the lore lines
item.tooltip(false)             // List<Text>: the whole tooltip, as drawn
```

`tooltip(true)` is the F3+H version, with the item id and durability numbers.

Matching on lore is the usual way to read a shop:

```kotlin
val price = inventory.held().lore()
    .map { it.string() }
    .firstOrNull { it.startsWith("Price:") }
```

## Attributes and components

```kotlin
for (modifier in item.attributeModifiers()) {
    modifier.attribute()        // "minecraft:attack_damage"
    modifier.value()
    modifier.operation()        // "add_value", "add_multiplied_base", "add_multiplied_total"
    modifier.slot()             // "mainhand", "head", "any"...
}
```

Shulker boxes and bundles carry their contents with them, so you can read a box without opening it:

```kotlin
val inside = item.containerItems()
chat.print("box holds " + inside.size + " stacks")
```

Everything an item stores lives in a **component**. The ones worth having are above; for anything else there is a generic way in:

```kotlin
item.hasComponent("minecraft:trim")
item.component("minecraft:dyed_color")   // a readable dump, shape depends on the component
```

`component(...)` hands back a string because every component has its own shape. Use it to check and to log, not to parse.

Servers that keep private data on an item use custom data instead:

```kotlin
item.nbt()                      // the whole custom-data tag as text
item.customData("shopId")       // one string key out of it, or null
```

Always check for empty: `inventory.item(...)` never returns `null`, it returns an empty item.

```kotlin
val held = inventory.held()
if (held.empty()) return
```

## Moving things around

```kotlin
inventory.click(slot, false)               // false is the left button
inventory.shiftClick(slot)
inventory.swap(slot, Slot.hotbar(8))
inventory.drop(slot, true)                 // true drops the whole stack
inventory.dropHeld(false)
```

When you need several actions, group them into one batch so the server is less likely to see a desync:

```kotlin
val hotbar = Slot.hotbar(8)

inventory.batch {
    it.swap(slot, hotbar)
    it.delay(1)
    it.swap(Slot.armor(ArmorSlot.CHESTPLATE), hotbar)
    it.delay(1)
    it.swap(slot, hotbar)
}
```

`delay(ticks)` puts a gap between two steps of the batch, measured in **game ticks**. The batch does not block: your handler returns immediately and the queue keeps draining over the following ticks. A three-step armour swap needs those gaps — without them the server sees three clicks in one tick and the middle one lands on a slot whose contents it has not updated yet.

`onFinish { ... }` runs your code once the batch has finished — every step has gone out and the inventory has been closed. Use it to chain the next move instead of polling `busy()` every tick:

```kotlin
inventory.batch {
    it.swap(slot, hotbar)
    it.onFinish {
        // the batch is done, the inventory is closed
    }
}
```

An empty batch is dropped whole, together with its `onFinish`.

## Do not trip over yourself

`inventory.busy()` tells you the client is moving items right now — yours or someone else's. While it is `true`, stay out of the way:

```kotlin
on<PrePlayerTickEvent> {
    if (inventory.busy()) return@on
    // now it is safe to move things
}
```

Switching the held slot and picking the best armor are their own topic — see [Slots and armor](../actions/slots.md).
