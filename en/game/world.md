# World and blocks

`world` is the current world. It only works while you are in a game, so wrap things in `whenInGame` or check `inGame`.

## Which world

```kotlin
world.dimension()        // "minecraft:overworld" and friends
world.timeTicks()        // how long the world has existed
world.dayTimeTicks()     // time of day, 0..24000
world.raining()
world.thundering()
world.rainGradient()     // 0..1, how hard it is raining right now
world.thunderGradient()  // 0..1, the same for a thunderstorm
```

`raining()` flips the moment weather starts; the gradient climbs over a few seconds, so it is the one to fade an effect with.

## Blocks

A block comes from coordinates or from a point:

```kotlin
val under = world.block(player.x().toInt(), player.y().toInt() - 1, player.z().toInt())
val here = world.block(player.position())
```

What you can ask about it:

```kotlin
under.id()               // "minecraft:stone"
under.name()             // its in-game name
under.x()  under.y()  under.z()
under.position()
under.box()              // its box in the world

under.air()
under.solid()
under.liquid()
under.opaque()
under.blocksMovement()
under.replaceable()      // your own block can go here: air, water, grass

under.luminance()        // how much light it emits
under.hardness()         // how long it takes to break
under.blastResistance()
under.slipperiness()     // ice is slippery

under.fullCube()
under.hasCollision()
under.hasBlockEntity()   // a chest, a sign, a shulker
under.emitsRedstone()
under.toolRequired()     // breaking it bare-handed drops nothing
under.burnable()
under.randomTicks()
under.pistonBehavior()   // "normal", "destroy", "block", "push_only"
under.velocityMultiplier()      // soul sand slows you
under.jumpVelocityMultiplier()  // honey shortens the jump
under.opacity()          // how much light it swallows
under.mapColor()         // its colour on a map, 0xAARRGGBB
```

## Block state and tags

A block id tells you *what* it is; the state tells you *how* it is placed. That is where "is this chest open", "which way does the stair face" and "how full is this cauldron" live:

```kotlin
val block = world.block(position)

block.properties()          // every property as name -> value, both strings
block.property("facing")    // one of them, or null if the block has no such property
```

```kotlin
if (block.property("open") == "true") {
    chat.print("the trapdoor is open")
}
```

Values are always strings, because the set of properties depends on the block — `"true"`, `"north"`, `"5"`. Compare them as text, do not expect numbers.

Tags group blocks the way the game groups them:

```kotlin
block.tags()                     // every tag the block is in
block.hasTag("mineable/pickaxe")
block.hasTag("minecraft:logs")   // the prefix is optional
```

## Block entities

A chest, a sign, a vault, a shulker — those blocks carry data the block id and the state say nothing about. `blockEntity()` gets you to it, and returns `null` for an ordinary block:

```kotlin
val vault = world.block(position).blockEntity() ?: return

vault.type()          // "minecraft:vault"
vault.x()  vault.y()  vault.z()
vault.position()
vault.displayItem()   // the item it is showing you, or null
vault.nbt()           // everything the server sent about it
```

`displayItem()` is the reward spinning inside a trial vault, and the buried loot inside suspicious sand or gravel:

```kotlin
on<ClientTickEvent> {
    whenInGame {
        val hit = raycast.crosshair(5.0)
        if (hit !is Hit.OnBlock) return@whenInGame
        val loot = world.block(hit.blockX(), hit.blockY(), hit.blockZ())
            .blockEntity()?.displayItem() ?: return@whenInGame
        chat.print("this vault holds ${loot.name()}")
    }
}
```

It is an ordinary [item](inventory.md), so `id()`, `name()`, `isA("heavy_core")`, `enchantments()` and the rest all work on it.

For anything else there is `nbt()` — the raw data as a string, exactly as Minecraft writes it:

```kotlin
val sign = world.block(position).blockEntity() ?: return
if (sign.type() == "minecraft:sign") {
    chat.print(sign.nbt())
}
```

## Finding block entities around you

Rather than walking coordinates yourself, ask for a whole volume at once:

```kotlin
val vaults = world.blockEntitiesIn(Box.around(player.position(), 16.0))
    .filter { it.type() == "minecraft:vault" }

for (vault in vaults) {
    val loot = vault.displayItem() ?: continue
    if (loot.isA("heavy_core")) {
        chat.print("heavy core at ${vault.x()} ${vault.y()} ${vault.z()}")
    }
}
```

A block counts when its centre is inside the box. The box may not span more than 32×32 chunks — far more than you can have loaded anyway.

Three things to keep in mind.

**You only see what the server sent you.** A vault's spinning item is sent because your client has to draw it; a chest's contents are not sent until you open the chest, so a chest block entity is real but empty. That is a Minecraft rule, not a limit of this API — nothing in the client knows more than this.

**Both calls work only on the client thread.** From a packet handler wrap them in `onClientThread { }`, or you get a `ScriptThreadException`. If all you want is the moment a block entity changes, `S2CBlockEntityUpdatePacket` carries the same data in its `nbt()` and is safe to read right there — see [Packets](../actions/packets.md).

**Nothing here reaches unloaded chunks.** Outside your render distance there is no block entity to read.

## Fluids

```kotlin
val fluid = world.block(position).fluid() ?: return

fluid.id()        // "minecraft:water"
fluid.level()     // 0..8, how full the block is
fluid.still()     // a source block rather than flowing
fluid.height()    // 0..1, what the fluid actually reaches
```

`fluid()` is `null` for a dry block. Water inside a waterlogged stair counts — the block is `minecraft:oak_stairs` and the fluid is water.

A couple more useful things about space:

```kotlin
world.lightLevel(x, y, z)      // light level
world.blockLight(x, y, z)      // from torches and lamps only
world.skyLight(x, y, z)        // from the sky only
world.biome(position)          // "minecraft:plains"
world.topY(x, z)               // height of the topmost block
world.collisionsIn(box)        // every collision inside a box
world.isFree(box)              // is it empty in there
```

`lightLevel` is what the game actually renders. Split it when you care *why* a spot is dark: `skyLight` is 0 underground at any time of day, `blockLight` is 0 wherever nobody put a torch.

## Boxes

A `Box` is a rectangular volume. Entities and blocks have one, and you can build your own:

```kotlin
val around = Box.around(player.position(), 8.0)

around.center()
around.sizeX()
around.expand(1.0)             // grow in every direction
around.contract(0.5)
around.offset(0.0, 1.0, 0.0)
around.contains(point)
around.intersects(other)
```

## Entities around you

```kotlin
world.entities()                                    // everything
world.entities(filters.player())                    // with a filter
world.entitiesIn(Box.around(player.position(), 8.0))
world.entitiesNear(player.position(), 8.0, filters.attackable())
world.players()
world.entityCount()

world.entityById(id)
world.entityByUuid(uuid)
world.entityByName("Notch")
world.nearestEntity(player.position(), 16.0, filters.monster())
```

Anything returning a single entity can return `null` — meaning nothing matched.

Filters and what you can ask an entity are in [Entities and filters](entities.md).

## Removing an entity

You can take an entity out of the world you are looking at:

```kotlin
world.removeEntity(entity)       // by entity
world.removeEntity(entityId)     // by id, when that is all you have
```

It returns `true` when something was actually removed, `false` when the entity was already gone — and `false` for your own player, which is never removed.

This is your copy of the world and nothing else. The server still has the entity, still sends updates about it, and other players see it exactly where it was. Anything the entity was doing to you — a hitbox in the way, an explosion, damage — keeps happening; you have only stopped drawing and tracking it locally. If the server sends the entity again, on a chunk reload or a teleport, it comes back.

```kotlin
on<ClientTickEvent> {
    whenInGame {
        for (entity in world.entities(filters.item())) {
            world.removeEntity(entity)
        }
    }
}
```

Two things to keep in mind. It only works on the client thread — from a packet handler wrap it in `onClientThread { }`, or you get a `ScriptThreadException`. And removing an entity fires `EntityRemoveEvent`, your own handler included, so do not remove from inside that handler unless you mean it.

## Bringing hidden entities back

```kotlin
world.unhideEntities()
```

Undoes every `entity.hidden(true)` in one call — see [Hiding one](entities.md#hiding-one). Hiding is not owned by the script that did it, so this is what belongs in your `onDisable`. Nothing about the entities themselves changes: they never went anywhere, you were only not drawing them.

## Sounds and particles

```kotlin
world.playSound("minecraft:entity.experience_orb.pickup", 1f, 1f)
world.playSound("minecraft:block.anvil.land", position, 1f, 0.8f)

world.spawnParticle("minecraft:flame", position, Vec.ZERO)
```

Without a position the sound plays where you are. Volume and pitch are ordinary Minecraft values: `1f` is normal, `2f` higher, `0.5f` lower.

All of this is seen and heard **by you only** — the server knows nothing about it.
