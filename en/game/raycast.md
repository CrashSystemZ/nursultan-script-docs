# Rays and the crosshair

`raycast` answers "what is in front of me". It is also how you check whether a point is visible and where exactly a hit will land.

## What is under the crosshair

The common one is grabbing the entity you are looking at:

```kotlin
val target = raycast.entityAtCrosshair(player.entityReachBlocks()) ?: return@on
```

You can filter out the ones you do not want right away:

```kotlin
val target = raycast.entityAtCrosshair(6.0) { !it.isAlly() && it.alive() } ?: return@on
```

You get `null` when nothing matching is under the crosshair.

## The hit

The general form is `crosshair(...)`, which tells you what the ray ran into: a block, an entity, or nothing.

```kotlin
val hit = raycast.crosshair(player.blockReachBlocks())

hit.position()            // where it landed
hit.distance()            // how far that is

when {
    hit.isBlock() -> {
        val block = hit as Hit.OnBlock
        chat.print("block " + block.blockX() + " " + block.blockY() + " " + block.blockZ())
        chat.print("side " + block.side())
    }
    hit.isEntity() -> {
        val entity = (hit as Hit.OnEntity).entity()
        chat.print("entity " + entity.name())
    }
    hit.isMiss() -> chat.print("miss")
}
```

In Kotlin a `when` over the type reads better:

```kotlin
when (val hit = raycast.crosshair(5.0)) {
    is Hit.OnBlock -> chat.print("block " + hit.side())
    is Hit.OnEntity -> chat.print("entity " + hit.entity().name())
    is Hit.None -> {}
}
```

The long form gives you more control:

```kotlin
raycast.crosshair(6.0, true) { it.isPlayer() }   // true includes blocks
```

## A ray that is not the crosshair

Sometimes you want to look not where you are looking, but where you are about to:

```kotlin
val aim = rotations.lookAt(target.position())
val hit = raycast.from(aim, 6.0, true) { !it.isAlly() }
```

That is how you check a block is not in the way before turning your head.

## Blocks only

```kotlin
val hit = raycast.blocks(from, to, RaycastShape.COLLIDER, FluidHandling.NONE)
```

| Shape | What counts |
|---|---|
| `COLLIDER` | the collision shape |
| `OUTLINE` | the outline, as used for highlighting |
| `VISUAL` | the visible shape |

| Fluids | What blocks the ray |
|---|---|
| `NONE` | water and lava do not |
| `SOURCE_ONLY` | source blocks only |
| `ANY` | any fluid |

## Is a point visible

```kotlin
if (raycast.canSee(player.eyePosition(), target.position())) {
    // no blocks between us
}
```

## Where exactly will I land

`hitOn` gives you the point on an entity's box the ray runs into, or `null` on a miss:

```kotlin
val point = raycast.hitOn(target, player.eyePosition(), aimPoint)
```

A ready-made way to pick a good hit point is [`combat.attackPoint`](../actions/interaction.md).

## About performance

Rays are not free. One or two per tick is fine; dozens across every entity every tick is not. Narrow things down by distance with `world.entitiesNear(...)` first, then shoot rays at what is left.
