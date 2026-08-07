# Auto soup

Eats mushroom stew when your health drops: swaps to it in the hotbar, right clicks, and hands the slot back. Refills the hotbar from the rest of the inventory and can throw the empty bowls away. An example of held slots, inventory batches and staying out of the client's way.

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

## Things worth noticing

**The swap and the click are not the same tick.** `slots.using { }` would do both at once; here the slot is taken with `slots.select(...)`, held for `holdTicks`, and only then right clicked. That is what the manual form of a held slot is for, and `restoreWhenSafe()` gives the slot back without cutting the use short.

**The hold is re-checked every tick.** If the stew leaves the slot while it is held — eaten, dropped, moved by something else — `isA(SOUP)` fails and the slot goes straight back. A held slot that nobody releases is the one bug this shape can produce, which is also why `onDisable` releases it.

**A random gap between meals.** `rangeSlider` gives a from/to pair and `rollDelay()` picks a number inside it after every soup, so the interval is never the same twice.

**Item moves go through `inventory.batch`.** Each batch does at most `perBatch` moves with `batch.delay(gap)` between them. Clicking thirty-six slots inside one tick is exactly the thing a server notices; this spreads them out and lets the next tick decide whether to continue.

**Dropping bowls keeps a timer of its own on top of that.** With a zero gap a batch is over almost as soon as it started, and `inventory.busy()` stops guarding — the next tick would queue another one, plus a full inventory scan looking for bowls. `sinceDrop` puts at least a tick between batches. Refilling needs no such timer: it stops itself the moment there is no stock or no space, and it runs when you are already low, where waiting is the wrong answer.

**Moves only happen on screens where they are legal.** `game.screenKind()` has to be `NONE`, `INVENTORY` or `CHAT` — with a chest open, the click ids belong to that container, not to your inventory.

**`inventory.busy()` first, everything else after.** It is `true` while the client is already moving items, yours or another module's. Starting a batch on top of that is how two features fight over the same slot.

**`keepItemUse` is there because eating is not the only thing a right click does.** Without it the script would cut off a bow you were drawing or a potion you were drinking.
