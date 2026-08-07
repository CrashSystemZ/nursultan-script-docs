# Interaction

`interaction` is everything the player does with their hands: hit, use, place, break.

## Hitting

```kotlin
interaction.attack(target)        // hit a specific entity
interaction.attackCrosshair()     // hit whatever is under the crosshair
interaction.swing(Hand.MAIN_HAND) // just swing
```

Before hitting you normally check the cooldown, or the damage is pitiful:

```kotlin
on<ClientTickEvent> {
    if (!inGame) return@on
    val target = raycast.entityAtCrosshair(player.entityReachBlocks()) { !it.isAlly() } ?: return@on
    if (player.attackCooldown() < 0.9f) return@on
    interaction.attack(target)
}
```

## Where to hit

`combat.attackPoint(...)` picks a point on the target worth aiming at:

```kotlin
val point = combat.attackPoint(target)
val point = combat.attackPoint(target, AttackPoint.MULTI_POINT, 3.0, false)
```

| Mode | What it does |
|---|---|
| `CENTER` | the centre of the box |
| `NEAREST` | the closest point to you |
| `MULTI_POINT` | tries several points and takes a reachable one |
| `TRIANGLE` | like MULTI_POINT, with the points spread differently |

The last two arguments are the maximum distance and whether hitting through walls is allowed.

`combat` can also mark a target so the client highlights it:

```kotlin
combat.markTarget(living, 15)    // for 15 ticks
combat.target()                  // who the client currently considers the target
```

## Using an item

```kotlin
interaction.useItem(Hand.MAIN_HAND)
interaction.useItem(Hand.OFF_HAND)
interaction.stopUsingItem()      // release right click: fire, throw the trident
interaction.useCrosshair()       // right click on whatever is under the crosshair
interaction.swapHands()          // swap hands
```

For example, releasing a trident by itself once it is fully charged:

```kotlin
on<ClientTickEvent> {
    if (!player.usingItem()) return@on
    if (!inventory.held().isA("trident")) return@on
    if (player.itemUseTicks() < 10) return@on
    interaction.stopUsingItem()
}
```

## Eating, drinking, charging

`useItem` only *starts* the use. The game releases right click for you on the very next tick unless the key is physically held, so a potion started from a script is aborted before it is drunk. Hold it yourself by cancelling the release while you want the use to continue:

```kotlin
var busy = false

on<StopUsingItemEvent> { e -> if (busy) e.cancel() }

fun drink() {
    interaction.useItem(Hand.MAIN_HAND)
    busy = true
}

on<ClientTickEvent> {
    if (busy && !player.usingItem()) {
        busy = false   // the item finished on its own and was consumed
    }
}
```

The use ends by itself when the item is consumed — `player.usingItem()` goes back to `false`, and that path does not go through the event. Always clear the flag: a stuck `busy` means right click can never be released again.

## Blocks

```kotlin
interaction.useBlock(x, y, z, Side.UP, Hand.MAIN_HAND)     // press a block
interaction.placeBlock(x, y, z, Side.UP, Hand.MAIN_HAND)   // place a block
```

Breaking takes three steps because it is a process, not one action:

```kotlin
interaction.startBreaking(x, y, z, Side.UP)

on<ClientTickEvent> {
    if (interaction.breakingBlock()) {
        val done = interaction.continueBreaking(x, y, z, Side.UP)
        if (done) {
            interaction.stopBreaking()
        }
    }
}
```

`continueBreaking` returns `true` when the block is gone. `breakingBlock()` tells you whether anything is being broken right now — including by the player themselves.

## Which side

`Side` is a face of a block: `UP`, `DOWN`, `NORTH`, `SOUTH`, `WEST`, `EAST`. When you shoot a [ray](../game/raycast.md), the side comes with the hit:

```kotlin
val hit = raycast.crosshair(player.blockReachBlocks())
if (hit is Hit.OnBlock) {
    interaction.placeBlock(hit.blockX(), hit.blockY(), hit.blockZ(), hit.side(), Hand.MAIN_HAND)
}
```

`side.opposite()` gives you the opposite face.
