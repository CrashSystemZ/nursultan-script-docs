# Rays and the crosshair

`raycast` is `game.raycast()`. Every method needs the client thread, a player and a world; distances are in blocks measured from the ray origin.

```kotlin
on<ClientTickEvent> {
    val hit = raycast.crosshair(4.5)
    when {
        hit.isBlock() -> chat.print("block " + (hit as Hit.OnBlock).side())
        hit.isEntity() -> chat.print("entity " + (hit as Hit.OnEntity).entity().name())
        hit.isMiss() -> chat.print("miss")
    }
}
```

## Casting

| Method | Type | Description |
|---|---|---|
| `raycast.crosshair(maxDistanceBlocks)` | `Hit` | crosshair ray including blocks and every hittable entity (throws `ScriptException` when `maxDistanceBlocks <= 0`) |
| `raycast.crosshair(maxDistanceBlocks, includeBlocks, entityFilter)` | `Hit` | same with a block toggle and an entity filter; null filter keeps all (throws `ScriptException` when `maxDistanceBlocks <= 0`) |
| `raycast.entityAtCrosshair(maxDistanceBlocks)` | `Entity?` | entity under the crosshair, blocks occlude |
| `raycast.entityAtCrosshair(maxDistanceBlocks, filter)` | `Entity?` | same with an entity filter |
| `raycast.from(rotation, maxDistanceBlocks, includeBlocks, entityFilter)` | `Hit` | same ray from the eyes along an arbitrary rotation (throws `ScriptException` when `rotation` is null or `maxDistanceBlocks <= 0`) |
| `raycast.blocks(from, to, shape, fluids)` | `Hit` | block-only ray between two points; null shape is `COLLIDER`, null fluids `NONE` |
| `raycast.canSee(from, to)` | `boolean` | true when no `COLLIDER` block interrupts the segment, fluids ignored |
| `raycast.hitOn(entity, from, to)` | `Vec?` | impact point on the hitbox expanded by its targeting margin, `from` when the segment starts inside |

Spectators and entities that cannot be hit are never returned, filter or not.
Off the client thread every method throws [`ScriptThreadException`](../extras/limits.md#exceptions); with no player or world, `ScriptStateException`. `Vec` and `Rotation` are on [Vectors, boxes, angles](math.md).

## The hit

### Hit

| Method | Type | Description |
|---|---|---|
| `hit.position()` | `Vec` | impact point in world coordinates |
| `hit.distance()` | `double` | blocks from the ray origin to `position()` |
| `hit.isBlock()` | `boolean` | result is `Hit.OnBlock` |
| `hit.isEntity()` | `boolean` | result is `Hit.OnEntity` |
| `hit.isMiss()` | `boolean` | result is `Hit.None` |

On a `blocks(...)` miss `position()` is `to`; when a crosshair trace returns nothing it is the ray origin and `distance()` is 0.

### Hit.OnBlock, Hit.OnEntity

| Method | Type | Description |
|---|---|---|
| `Hit.OnBlock.blockX()` | `int` | x coordinate of the hit block |
| `Hit.OnBlock.blockY()` | `int` | y coordinate of the hit block |
| `Hit.OnBlock.blockZ()` | `int` | z coordinate of the hit block |
| `Hit.OnBlock.side()` | `Side` | face of the block that was hit |
| `Hit.OnEntity.entity()` | `Entity` | the entity the ray hit |

`Hit` is sealed over `OnBlock`, `OnEntity` and `None`; `Hit.None` adds no members of its own.

## Which shape

### RaycastShape

| Constant | Description |
|---|---|
| `COLLIDER` | collision shape |
| `OUTLINE` | outline shape, the one used for highlighting |
| `VISUAL` | visual shape |

### FluidHandling

| Constant | Description |
|---|---|
| `NONE` | fluids never stop the ray |
| `SOURCE_ONLY` | only source fluid blocks stop the ray |
| `ANY` | any fluid, flowing included, stops the ray |

## Sides

### Side

| Constant | Description |
|---|---|
| `DOWN` | -Y face |
| `UP` | +Y face |
| `NORTH` | -Z face |
| `SOUTH` | +Z face |
| `WEST` | -X face |
| `EAST` | +X face |

### Side methods

| Method | Type | Description |
|---|---|---|
| `side.opposite()` | `Side` | face on the other end of the same axis |

`Side` is the same value [`interaction.useBlock` / `placeBlock` / `startBreaking`](../actions/interaction.md) take.
