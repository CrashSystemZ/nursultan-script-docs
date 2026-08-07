# Автосуп

Ест грибной суп, когда здоровье падает: переключается на него в хотбаре, кликает ПКМ и отдаёт слот обратно. Пополняет хотбар из остального инвентаря и умеет выбрасывать пустые миски. Пример удержанных слотов, инвентарных батчей и умения не лезть клиенту под руку.

```kotlin
name("AutoSoup")
description("Swaps to soup in the hotbar, right clicks it and swaps back when health drops")

val health = slider("Health", 12, 1, 19)
val delayTicks = rangeSlider("Delay in ticks", 3f, 5f, 0f, 20f)
val holdTicks = slider("Hold before eating in ticks", 2, 0, 20)
val keepItemUse = checkBox("Don't interrupt item use", true)
val refill = checkBox("Refill hotbar", true)
val dropBowls = checkBox("Drop bowls", false)
val perBatch = slider("Items per batch", 2, 1, 9)
val batchDelay = slider("Batch delay in ticks", 1, 0, 10)

val SOUP = "minecraft:mushroom_stew"
val BOWL = "minecraft:bowl"

val sinceSoup = timer()
val sinceHold = timer()
val sinceDrop = timer()

var nextDelay = 0
var refilling = false
var held: HeldSlot? = null

fun releaseHold() {
    held?.restoreWhenSafe()
    held = null
}

fun rollDelay() {
    val from = delayTicks.from().toInt()
    val to = delayTicks.to().toInt()
    nextDelay = if (to <= from) from else Random.nextInt(from, to)
}

fun canMoveItems(): Boolean {
    val screen = game.screenKind()
    return screen == ScreenKind.NONE || screen == ScreenKind.INVENTORY || screen == ScreenKind.CHAT
}

fun hotbarSpace(): List<Slot> = (0 until Slot.HOTBAR_SIZE)
    .map { Slot.hotbar(it) }
    .filter { slot ->
        val item = inventory.item(slot)
        item.empty() || item.isA(BOWL)
    }

fun refillStep(): Boolean {
    if (!canMoveItems()) return false

    val space = hotbarSpace()
    val stock = inventory.findAll { it.isA(SOUP) }.filter { !it.inHotbar() }
    if (space.isEmpty() || stock.isEmpty()) {
        refilling = false
        return false
    }

    val moves = minOf(perBatch.intValue(), space.size, stock.size)
    val gap = batchDelay.intValue()
    inventory.batch { batch ->
        for (i in 0 until moves) {
            if (i > 0 && gap > 0) batch.delay(gap)
            batch.swap(stock[i], space[i])
        }
    }
    return true
}

fun dropStep() {
    if (!canMoveItems()) return
    if (!sinceDrop.passed(maxOf(batchDelay.intValue(), 1))) return

    val bowls = inventory.findAll { it.isA(BOWL) }
    if (bowls.isEmpty()) return

    val moves = minOf(perBatch.intValue(), bowls.size)
    val gap = batchDelay.intValue()
    inventory.batch { batch ->
        for (i in 0 until moves) {
            if (i > 0 && gap > 0) batch.delay(gap)
            batch.drop(bowls[i], true)
        }
    }
    sinceDrop.reset()
}

onEnable {
    sinceSoup.reset()
    nextDelay = 0
    refilling = false
}

onDisable {
    releaseHold()
}

on<PrePlayerTickEvent> {
    if (!player.alive()) {
        releaseHold()
        return@on
    }
    if (inventory.busy()) return@on

    val soup = inventory.findInHotbar(SOUP)
    val handFree = !keepItemUse.value() || !player.usingItem()

    if (held == null
        && soup.found()
        && handFree
        && player.health() <= health.intValue()
        && sinceSoup.passed(nextDelay)
    ) {
        held = slots.select(soup)
        sinceHold.reset()
    }

    val holding = held
    if (holding != null) {
        if (!inventory.item(Slot.hotbar(holding.slot())).isA(SOUP)) {
            releaseHold()
        } else if (sinceHold.passed(holdTicks.intValue())) {
            interaction.useItem(Hand.MAIN_HAND)
            releaseHold()
            rollDelay()
            sinceSoup.reset()
        }
        return@on
    }

    if (!refill.value()) {
        refilling = false
    } else if (!soup.found()) {
        refilling = true
    }

    if (refilling && refillStep()) return@on

    if (dropBowls.value()) {
        dropStep()
    }
}
```

## Что тут стоит заметить

**Подмена слота и клик — не один тик.** `slots.using { }` сделал бы и то и другое разом; здесь слот берётся через `slots.select(...)`, держится `holdTicks` и только потом кликается. Ровно для этого и нужна ручная форма удержанного слота, а `restoreWhenSafe()` возвращает слот, не обрывая использование.

**Удержание перепроверяется каждый тик.** Если суп ушёл из слота, пока его держат — съели, выбросили, переложили чем-то ещё, — `isA(SOUP)` не сходится и слот сразу возвращается. Удержанный слот, который никто не отпустил, — единственный баг, который эта схема умеет порождать; поэтому же `onDisable` его отпускает.

**Случайная пауза между приёмами.** `rangeSlider` даёт пару «от/до», а `rollDelay()` после каждого супа выбирает число внутри неё — интервал никогда не повторяется дважды.

**Перекладывание идёт через `inventory.batch`.** В одном батче не больше `perBatch` перестановок, между ними `batch.delay(gap)`. Прокликать тридцать шесть слотов за один тик — это ровно то, что сервер замечает; так они растянуты, а продолжать или нет, решает следующий тик.

**У выброса мисок поверх этого свой таймер.** При нулевом промежутке батч заканчивается почти сразу, и `inventory.busy()` перестаёт прикрывать — следующий тик поставил бы в очередь ещё один, да ещё и с полным сканом инвентаря в поисках мисок. `sinceDrop` держит между батчами хотя бы тик. Пополнению такой таймер не нужен: оно само останавливается, как только кончился запас или место, и работает, когда ты уже на низком здоровье, — там ждать неправильно.

**Двигать предметы можно только на тех экранах, где это законно.** `game.screenKind()` обязан быть `NONE`, `INVENTORY` или `CHAT`: с открытым сундуком id кликов принадлежат этому контейнеру, а не твоему инвентарю.

**Сначала `inventory.busy()`, потом всё остальное.** Он `true`, пока клиент уже перекладывает предметы — твои или чужого модуля. Начать батч поверх этого — верный способ подраться за один слот.

**`keepItemUse` есть потому, что ПКМ — это не только еда.** Без него скрипт обрывал бы натянутый лук или недопитое зелье.
