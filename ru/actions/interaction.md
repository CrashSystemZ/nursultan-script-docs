# Взаимодействие

`interaction` — это всё, что игрок делает руками: бить, использовать, ставить, ломать.

## Бить

```kotlin
interaction.attack(target)       // ударить конкретную сущность
interaction.attackCrosshair()    // ударить то, что под прицелом
interaction.swing(Hand.MAIN_HAND) // просто махнуть рукой
```

Перед ударом обычно проверяют откат — иначе урон будет копеечный:

```kotlin
on<ClientTickEvent> {
    if (!inGame) return@on
    val target = raycast.entityAtCrosshair(player.entityReachBlocks()) { !it.isAlly() } ?: return@on
    if (player.attackCooldown() < 0.9f) return@on
    interaction.attack(target)
}
```

## Куда бить

`combat.attackPoint(...)` подбирает точку на цели, в которую имеет смысл целиться:

```kotlin
val point = combat.attackPoint(target)
val point = combat.attackPoint(target, AttackPoint.MULTI_POINT, 3.0, false)
```

| Режим | Что делает |
|---|---|
| `CENTER` | центр коробки |
| `NEAREST` | ближайшая к тебе точка |
| `MULTI_POINT` | перебирает несколько точек и берёт достижимую |
| `TRIANGLE` | как MULTI_POINT, но точки распределены иначе |

Последние два аргумента — максимальная дистанция и «можно ли бить сквозь стены».

Ещё `combat` умеет помечать цель, чтобы её подсветил клиент:

```kotlin
combat.markTarget(living, 15)    // на 15 тиков
combat.target()                  // кого клиент считает целью сейчас
```

## Использовать предмет

```kotlin
interaction.useItem(Hand.MAIN_HAND)
interaction.useItem(Hand.OFF_HAND)
interaction.stopUsingItem()      // отпустить ПКМ: выстрелить, кинуть трезубец
interaction.useCrosshair()       // ПКМ по тому, что под прицелом
interaction.swapHands()          // поменять руки местами
```

Пример: отпускать заряженный трезубец сам, когда он полностью заряжен.

```kotlin
on<ClientTickEvent> {
    if (!player.usingItem()) return@on
    if (!inventory.held().isA("trident")) return@on
    if (player.itemUseTicks() < 10) return@on
    interaction.stopUsingItem()
}
```

## Есть, пить, заряжать

`useItem` только **начинает** использование. Уже на следующем тике игра сама отпускает ПКМ, если кнопка не зажата физически, — поэтому зелье, начатое из скрипта, обрывается, так и не выпившись. Держи его сам, отменяя отпускание, пока использование должно продолжаться:

```kotlin
var busy = false

on<StopUsingItemEvent> { e -> if (busy) e.cancel() }

fun drink() {
    interaction.useItem(Hand.MAIN_HAND)
    busy = true
}

on<ClientTickEvent> {
    if (busy && !player.usingItem()) {
        busy = false   // предмет доиспользовался сам и был съеден
    }
}
```

Использование заканчивается само, когда предмет израсходован: `player.usingItem()` становится `false`, и этот путь через событие не идёт. Флаг обязательно сбрасывай — залипший `busy` означает, что ПКМ уже никогда не отпустится.

## Блоки

```kotlin
interaction.useBlock(x, y, z, Side.UP, Hand.MAIN_HAND)     // нажать на блок
interaction.placeBlock(x, y, z, Side.UP, Hand.MAIN_HAND)   // поставить блок
```

Ломание идёт в три шага, потому что это процесс, а не одно действие:

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

`continueBreaking` возвращает `true`, когда блок сломан. `breakingBlock()` говорит, ломается ли что-то прямо сейчас — в том числе руками игрока.

## Куда какой стороной

`Side` — это сторона блока: `UP`, `DOWN`, `NORTH`, `SOUTH`, `WEST`, `EAST`. Когда стреляешь [лучом](../game/raycast.md), сторона приходит вместе с попаданием:

```kotlin
val hit = raycast.crosshair(player.blockReachBlocks())
if (hit is Hit.OnBlock) {
    interaction.placeBlock(hit.blockX(), hit.blockY(), hit.blockZ(), hit.side(), Hand.MAIN_HAND)
}
```

`side.opposite()` даёт противоположную сторону.
