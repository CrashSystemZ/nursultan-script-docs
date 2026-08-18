# World and blocks

`world` is `game.world()`. Every call throws `ScriptStateException` when there is no world loaded; a block read is a snapshot taken at call time.

```kotlin
on<ClientTickEvent> {
    whenInGame {
        val under = world.block(player.x().toInt(), player.y().toInt() - 1, player.z().toInt())
        val fluid = under.fluid()
        val nearby = world.entitiesNear(player.position(), 8.0, filters.attackable())
        chat.print("${under.id()} ${fluid?.id() ?: "dry"} ${nearby.size}")
    }
}
```

## The world itself

| Method | Type | Description |
|---|---|---|
| `world.dimension()` | `String` | dimension registry key, e.g. `minecraft:overworld` |
| `world.timeTicks()` | `long` | total world age in ticks |
| `world.dayTimeTicks()` | `long` | time of day in ticks, cycle length 24000 |
| `world.raining()` | `boolean` | rain active in this world |
| `world.thundering()` | `boolean` | thunderstorm active in this world |
| `world.rainGradient()` | `float` | rain strength 0..1 |
| `world.thunderGradient()` | `float` | thunder strength 0..1 |
| `world.biome(position)` | `String?` | biome registry id at the floored position |
| `world.topY(x, z)` | `int` | y above the highest non-air block, bottom y when unloaded |

## Reading a block

| Method | Type | Description |
|---|---|---|
| `world.block(x, y, z)` | `Block` | block state snapshot, `void_air` outside world limits |
| `world.block(position)` | `Block` | same, `Vec` coordinates floored |
| `block.id()` | `String` | block registry id, e.g. `minecraft:stone` |
| `block.name()` | `String` | localized block name |
| `block.x()` | `int` | block x coordinate |
| `block.y()` | `int` | block y coordinate |
| `block.z()` | `int` | block z coordinate |
| `block.position()` | `Vec` | block corner coordinates |
| `block.box()` | `Box` | the 1×1×1 cube at this position, not the block shape |

`Vec` and `Box` are documented on [Vectors, boxes, angles](math.md).

## Block state

| Method | Type | Description |
|---|---|---|
| `block.air()` | `boolean` | state is air |
| `block.solid()` | `boolean` | state is solid |
| `block.liquid()` | `boolean` | state is a liquid block |
| `block.opaque()` | `boolean` | state is opaque |
| `block.blocksMovement()` | `boolean` | state blocks movement |
| `block.replaceable()` | `boolean` | a placed block can replace this state |
| `block.fullCube()` | `boolean` | state is a full cube at this position |
| `block.hasCollision()` | `boolean` | collision shape at this position is non-empty |
| `block.luminance()` | `int` | emitted light 0..15 |
| `block.opacity()` | `int` | light attenuation 0..15 |
| `block.hardness()` | `float` | breaking hardness at this position, -1 for unbreakable |
| `block.blastResistance()` | `float` | explosion resistance |
| `block.slipperiness()` | `float` | friction factor, 0.6 default, 0.98 for ice |
| `block.velocityMultiplier()` | `float` | horizontal movement multiplier |
| `block.jumpVelocityMultiplier()` | `float` | jump velocity multiplier |
| `block.emitsRedstone()` | `boolean` | state emits redstone power |
| `block.toolRequired()` | `boolean` | correct tool required for drops |
| `block.burnable()` | `boolean` | state is burnable |
| `block.randomTicks()` | `boolean` | state receives random ticks |
| `block.pistonBehavior()` | `String` | lowercase: normal, destroy, block, ignore, push_only |
| `block.mapColor()` | `int` | map colour as ARGB, alpha forced to 0xFF |

## Properties and tags

| Method | Type | Description |
|---|---|---|
| `block.properties()` | `Map<String, String>` | every state property as name -> value, unmodifiable, insertion-ordered |
| `block.property(name)` | `String?` | one property value by case-insensitive name |
| `block.tags()` | `List<String>` | block tag ids |
| `block.hasTag(tagId)` | `boolean` | block is in that tag, leading `#` and namespace optional (throws `ScriptException` when tagId is blank or unparsable) |

## Block entities

| Method | Type | Description |
|---|---|---|
| `block.hasBlockEntity()` | `boolean` | state declares a block entity |
| `block.blockEntity()` | `BlockEntity?` | block entity at this position (main thread only) |
| `world.blockEntitiesIn(box)` | `List<BlockEntity>` | block entities whose block centre is inside the box, immutable list (main thread only) (throws `ScriptException` when the box spans more than 32×32 chunks) |
| `blockEntity.type()` | `String` | block entity type registry id, e.g. `minecraft:vault` |
| `blockEntity.x()` | `int` | block x coordinate |
| `blockEntity.y()` | `int` | block y coordinate |
| `blockEntity.z()` | `int` | block z coordinate |
| `blockEntity.position()` | `Vec` | block coordinates |
| `blockEntity.displayItem()` | `Item?` | vault display item or brushable block item, null otherwise |
| `blockEntity.nbt()` | `String` | client-side nbt as a string, empty without a world |

A block entity carries only what the server already sent: a vault's display item is there, a closed chest's contents are not.
`displayItem()` returns an ordinary [item](inventory.md).

## Fluids

| Method | Type | Description |
|---|---|---|
| `block.fluid()` | `Fluid?` | fluid state at this position, waterlogging included, null when dry |
| `fluid.id()` | `String` | fluid registry id, e.g. `minecraft:water` |
| `fluid.level()` | `int` | fluid level 1..8, 8 for a source block |
| `fluid.still()` | `boolean` | source block rather than flowing |
| `fluid.height()` | `float` | rendered fluid height 0..1 |

## Light

| Method | Type | Description |
|---|---|---|
| `world.lightLevel(x, y, z)` | `int` | rendered light level 0..15, ambient darkness applied |
| `world.blockLight(x, y, z)` | `int` | block-source light 0..15 |
| `world.skyLight(x, y, z)` | `int` | sky-source light 0..15, no time-of-day adjustment |

## Entities in the world

| Method | Type | Description |
|---|---|---|
| `world.entityCount()` | `int` | number of loaded entities |
| `world.entities()` | `List<Entity>` | every loaded entity, own player included, fresh mutable list |
| `world.entities(filter)` | `List<Entity>` | loaded entities passing the filter, null filter keeps all |
| `world.entitiesIn(box)` | `List<Entity>` | entities whose hitbox intersects the box |
| `world.entitiesIn(box, filter)` | `List<Entity>` | same, filtered, null filter keeps all |
| `world.entitiesNear(origin, radiusBlocks, filter)` | `List<Entity>` | entities within a spherical radius in blocks (throws `ScriptException` when radiusBlocks <= 0) |
| `world.players()` | `List<PlayerEntity>` | loaded players, immutable list |
| `world.players(filter)` | `List<PlayerEntity>` | loaded players passing the filter, immutable list |
| `world.entityByName(name)` | `Entity?` | first entity whose display name matches, case-insensitive |
| `world.entityById(entityId)` | `Entity?` | entity with that network id |
| `world.entityByUuid(uuid)` | `Entity?` | linear scan for that uuid (throws `ScriptException` when the string is not a uuid) |
| `world.nearestEntity(origin, radiusBlocks, filter)` | `Entity?` | closest matching entity within the radius (throws `ScriptException` when radiusBlocks <= 0) |

Entity members and the ready-made filters are on [Entities and filters](entities.md).

## Collisions

| Method | Type | Description |
|---|---|---|
| `world.collisionsIn(box)` | `List<Box>` | boxes of hard-colliding entities in the box, block collisions excluded |
| `world.isFree(box)` | `boolean` | no colliding entity intersects the box, block collisions not checked |

## Removing and hiding

| Method | Type | Description |
|---|---|---|
| `world.removeEntity(entity)` | `boolean` | client-side removal by wrapper, false when null (main thread only) |
| `world.removeEntity(entityId)` | `boolean` | client-side removal by network id, false for own player (main thread only) |
| `world.unhideEntities()` | `void` | clears the [hidden](entities.md#hiding-one) flag on every loaded entity |

Removal is local: the server keeps the entity and resends it on a chunk reload, and each removal fires `EntityRemoveEvent`.

## Sounds and particles

| Method | Type | Description |
|---|---|---|
| `world.spawnParticle(particleId, position, velocity)` | `void` | client-only particle, null velocity means zero (throws `ScriptException` on an unknown particle) |
| `world.playSound(soundId, position, volume, pitch)` | `void` | client-only sound at a position, category PLAYERS (throws `ScriptException` on an unknown sound) |
| `world.playSound(soundId, volume, pitch)` | `void` | client-only sound at the listener, category PLAYERS (throws `ScriptException` on an unknown sound) |
