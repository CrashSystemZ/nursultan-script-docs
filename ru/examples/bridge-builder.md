# Строитель мостов

Ставит блок под ноги на ту грань, куда ты смотришь, и приседает, когда следующий шаг сбросил бы тебя с края. Пример того, как вместе работают предсказание движения, рейкаст из прицела и одолженный слот хотбара.

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
    .firstOrNull { inventory.item(it).buildable() }

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

## Что тут стоит заметить

**Два события, две задачи.** `MoveInputEvent` — это ввод, который игра вот-вот применит, поэтому safe walk живёт именно там: переписать `sneak` в нём — то же самое, что зажать shift самому. Постановка блока сидит в `PrePlayerTickEvent`, который идёт перед тиком, который тебя двигает.

**Safe walk спрашивает предсказание, а не блок под ногами.** `prediction.after(ticks, input)` прокручивает твоё движение без `forward`, `jump` и `sprint`, так что вопрос только про тот шаг вбок или назад, из-за которого всё началось. Останешься на земле — ничего не происходит; нет — `e.sneak(true)`. Смотреть на блок под ногами вместо этого — ошибаться на плитах, ступеньках и краях: предсказание считает настоящую физику.

**Блок кладётся рядом с гранью, а не в неё.** `hit.side()` — грань, в которую попал рейкаст, `offset(...)` превращает её в шаг, и получившаяся клетка обязана быть `replaceable()`, прежде чем что-то уйдёт на сервер. `world.isFree(cellBox)` — вторая половина этой проверки: моб или игрок, стоящий в клетке, мешает поставить блок ничуть не меньше, чем блок.

**`underFeet` — причина, по которой получается мост, а не стена.** Клетка считается, только если твой хитбокс нависает над ней по x и z, а её верх стоит на уровне ног или чуть ниже, но не глубже `maxDrop`. `predictTicks` прогоняет ту же проверку по коробке, в которой ты окажешься через пару тиков, — блок уже стоит, когда ты с края шагаешь.

**`slots.using(slot) { }` одалживает хотбар и возвращает его.** Блоки ищутся вопросом `buildable()` к каждому предмету хотбара — не `placeable()`, который спокойно отдаст тебе песок, лёд или паутину, — а подмена живёт ровно столько, сколько постановка и взмах, так что тик никогда не заканчивается с чужим предметом в руке.

**Пока клиент занят предметами, скрипт молчит.** `inventory.busy()` и `game.screenOpen()` держат его в стороне, пока ты или другой модуль что-то перекладываете.
