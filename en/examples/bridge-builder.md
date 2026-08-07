# Bridge builder

Places a block under your feet on the face you are aiming at, and sneaks when the next step would drop you off the edge. An example of movement prediction, a crosshair raycast and a borrowed hotbar slot working together.

```kotlin
name("BridgeBuilder")
description("Places a block under your feet at the face you aim at, and sneaks when a step would drop you")

val safeWalk = checkBox("Safe walk", true)
val safeWalkTicks = slider("Safe walk ticks", 3, 1, 10)
val predictTicks = slider("Predict ticks", 2, 0, 6)
val maxDrop = slider("Max drop", 1f, 0f, 3f, 0.1f)
val delayTicks = slider("Delay in ticks", 1, 0, 10)
val swing = checkBox("Swing hand", true)
val debug = checkBox("Debug", false)

val sincePlace = timer()

fun offset(side: Side): Triple<Int, Int, Int> = when (side) {
    Side.DOWN -> Triple(0, -1, 0)
    Side.UP -> Triple(0, 1, 0)
    Side.NORTH -> Triple(0, 0, -1)
    Side.SOUTH -> Triple(0, 0, 1)
    Side.WEST -> Triple(-1, 0, 0)
    Side.EAST -> Triple(1, 0, 0)
}

fun blockSlot(): Slot? = (0 until Slot.HOTBAR_SIZE)
    .map { Slot.hotbar(it) }
    .firstOrNull { inventory.item(it).placeable() }

fun underFeet(box: Box, x: Int, y: Int, z: Int): Boolean {
    if (box.maxX() <= x || box.minX() >= x + 1.0) return false
    if (box.maxZ() <= z || box.minZ() >= z + 1.0) return false
    val top = y + 1.0
    return top <= box.minY() + 0.01 && box.minY() - top <= maxDrop.value()
}

on<MoveInputEvent> { e ->
    if (!safeWalk.value() || !inGame) return@on
    if (!player.onGround() || player.flying() || player.riding()) return@on

    val backOrSide = e.backward() || e.left() || e.right()
    if (!backOrSide) return@on

    val input = e.toInput().copy(forward = false, jump = false, sprint = false)
    if (prediction.after(safeWalkTicks.intValue(), input).onGround()) return@on
    e.sneak(true)
}

on<PrePlayerTickEvent> {
    if (!player.alive() || player.flying() || player.riding()) return@on
    if (!sincePlace.passed(delayTicks.intValue())) return@on
    if (inventory.busy() || game.screenOpen()) return@on

    val hit = raycast.crosshair(player.blockReachBlocks())
    if (hit !is Hit.OnBlock) return@on

    val against = world.block(hit.blockX(), hit.blockY(), hit.blockZ())
    if (!against.blocksMovement()) return@on

    val (dx, dy, dz) = offset(hit.side())
    val x = hit.blockX() + dx
    val y = hit.blockY() + dy
    val z = hit.blockZ() + dz

    val cell = world.block(x, y, z)
    if (cell.blocksMovement() || !cell.replaceable()) return@on

    val cellBox = Box(x.toDouble(), y.toDouble(), z.toDouble(), x + 1.0, y + 1.0, z + 1.0)
    if (!world.isFree(cellBox)) {
        if (debug.value()) chat.print("bridge: $x $y $z blocked by an entity")
        return@on
    }

    val ticks = predictTicks.intValue()
    val soon = if (ticks <= 0) null else prediction.after(ticks).box()
    if (!underFeet(player.box(), x, y, z) && (soon == null || !underFeet(soon, x, y, z))) {
        return@on
    }

    val slot = blockSlot() ?: return@on

    slots.using(slot) {
        interaction.placeBlock(hit.blockX(), hit.blockY(), hit.blockZ(), hit.side(), Hand.MAIN_HAND)
        if (swing.value()) {
            interaction.swing(Hand.MAIN_HAND)
        }
    }
    sincePlace.reset()

    if (debug.value()) {
        chat.print("bridge: $x $y $z against ${hit.blockX()} ${hit.blockY()} ${hit.blockZ()} ${hit.side()}")
    }
}
```

## Things worth noticing

**Two events, two jobs.** `MoveInputEvent` is the input the game is about to use, so that is where safe walk belongs — rewriting `sneak` there is the same as you having held shift. Placement lives in `PrePlayerTickEvent`, which runs before the tick that moves you.

**Safe walk asks the prediction, not the block below.** `prediction.after(ticks, input)` replays your movement with `forward`, `jump` and `sprint` stripped out, so the question is only about the sideways or backwards step that triggered it. If you would still be on the ground, nothing happens; if not, `e.sneak(true)`. Looking at the block under your feet instead would get slabs, stairs and edges wrong — the prediction runs the real physics.

**The block goes next to the face you hit, not into it.** `hit.side()` is the face the raycast landed on, `offset(...)` turns it into a step, and the cell that comes out has to be `replaceable()` before anything is sent. `world.isFree(cellBox)` is the second half of that check: a mob or a player standing in the cell blocks the placement just as well as a block would.

**`underFeet` is why it builds a bridge and not a wall.** A cell only counts if your hitbox overhangs it in x and z and its top sits at or just below your feet, no further down than `maxDrop`. `predictTicks` runs the same test against the box you will occupy in a couple of ticks, so the block is already there when you step off.

**`slots.using(slot) { }` borrows the hotbar and gives it back.** The blocks are found by asking each hotbar item whether it is `placeable()`, and the swap lasts exactly as long as the place and the swing — you never end a tick holding something you did not choose.

**Nothing runs while the client is busy with items.** `inventory.busy()` and `game.screenOpen()` keep the script out of the way while you or another module is moving things around.
