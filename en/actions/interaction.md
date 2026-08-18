# Interaction

`interaction` is `game.interaction()` — attacking, using, swinging and breaking; every call except `breakingBlock()` runs on the client thread only. `combat` is `client.combat()` and supplies the aim point on a target's hitbox.

```kotlin
on<ClientTickEvent> {
    val target = raycast.entityAtCrosshair(player.entityReachBlocks()) ?: return@on
    if (player.attackCooldown() < 1f) return@on
    val point = combat.attackPoint(target, AttackPoint.MULTI_POINT, 3.0, false)
    rotations.apply(rotations.lookAt(point))
    interaction.attack(target)
}
```

## Attacking

| Method | Type | Description |
|---|---|---|
| `interaction.attack(target)` | `void` | attacks the entity and swings the main hand (main thread only) (throws `ScriptStateException` when the target is not alive) |
| `interaction.attackCrosshair()` | `void` | vanilla attack on whatever the crosshair hits (main thread only) |
| `interaction.swing(hand)` | `void` | plays the swing animation and sends the swing packet (main thread only) |

`Hand` constants are on [Inventory and items](../game/inventory.md#hands).

## Where to hit

| Method | Type | Description |
|---|---|---|
| `combat.attackPoint(target)` | [`Vec`](../game/math.md#vec) | `NEAREST` point, max distance = entity reach, through walls |
| `combat.attackPoint(target, mode, maxDistanceBlocks, throughWalls)` | [`Vec`](../game/math.md#vec) | aim point on the hitbox; distance and walls read only by `MULTI_POINT` and `TRIANGLE` (throws `ScriptException` when `maxDistanceBlocks` is 0 or less) |
| `combat.markTarget(target, timeoutTicks)` | `void` | marks the client's TargetESP target for that many ticks (main thread only) (throws `ScriptException` when `timeoutTicks` is 0 or less) |
| `combat.target()` | [`Entity`](../game/entities.md)`?` | current TargetESP target, null when there is none |

A null `mode` is treated as `NEAREST`; both `attackPoint` overloads throw `ScriptStateException` when the entity has left the world.
`markTarget` throws `ScriptStateException` when the target is not a living entity.

| Constant | Description |
|---|---|
| `CENTER` | geometric centre of the bounding box |
| `NEAREST` | closest surface point to the eyes, hitbox shrunk by 0.01 blocks |
| `MULTI_POINT` | best of an 11×11 grid over the visible faces, each candidate ray-checked |
| `TRIANGLE` | one point per visible face, ranked by reach then mouse delta |

## Using an item

| Method | Type | Description |
|---|---|---|
| `interaction.useItem(hand)` | `void` | starts using the item in that hand (main thread only) |
| `interaction.useCrosshair()` | `void` | vanilla use on whatever the crosshair hits (main thread only) |
| `interaction.stopUsingItem()` | `void` | releases the use: fires the bow, throws the trident (main thread only) |
| `interaction.swapHands()` | `void` | sends the offhand swap packet (main thread only) (throws `ScriptStateException` when not connected) |

The game calls `stopUsingItem` itself on the next tick while the use key is not physically held, so a use started from a script ends immediately unless `StopUsingItemEvent` is cancelled.

## Blocks

| Method | Type | Description |
|---|---|---|
| `interaction.useBlock(x, y, z, side, hand)` | `void` | interacts with the centre of that block face (main thread only) |
| `interaction.placeBlock(x, y, z, side, hand)` | `void` | identical to `useBlock`, delegates to it (main thread only) |
| `interaction.startBreaking(x, y, z, side)` | `void` | begins breaking that block (main thread only) |
| `interaction.continueBreaking(x, y, z, side)` | `boolean` | advances breaking, true when the block broke (main thread only) |
| `interaction.stopBreaking()` | `void` | cancels the current breaking progress (main thread only) |
| `interaction.breakingBlock()` | `boolean` | true while a block is being broken, false when out of world |

`Side` constants and `opposite()` are on [Rays and the crosshair](../game/raycast.md#sides).
