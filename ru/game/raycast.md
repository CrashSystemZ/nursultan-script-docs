# Лучи и прицел

`raycast` отвечает на вопрос «что там впереди». Им же проверяют, видно ли точку, и куда именно попадёт удар.

## Что под прицелом

Самое частое — взять сущность, на которую смотришь:

```kotlin
val target = raycast.entityAtCrosshair(player.entityReachBlocks()) ?: return@on
```

Можно сразу отсеять ненужных:

```kotlin
val target = raycast.entityAtCrosshair(6.0) { !it.isAlly() && it.alive() } ?: return@on
```

Вернётся `null`, если ничего подходящего под прицелом нет.

## Попадание

Более общий вариант — `crosshair(...)`, он говорит, во что упёрся луч: в блок, в сущность или ни во что.

```kotlin
val hit = raycast.crosshair(player.blockReachBlocks())

hit.position()            // точка попадания
hit.distance()            // сколько до неё

when {
    hit.isBlock() -> {
        val block = hit as Hit.OnBlock
        chat.print("блок " + block.blockX() + " " + block.blockY() + " " + block.blockZ())
        chat.print("сторона " + block.side())
    }
    hit.isEntity() -> {
        val entity = (hit as Hit.OnEntity).entity()
        chat.print("сущность " + entity.name())
    }
    hit.isMiss() -> chat.print("мимо")
}
```

В Kotlin это удобнее писать через `when` по типу:

```kotlin
when (val hit = raycast.crosshair(5.0)) {
    is Hit.OnBlock -> chat.print("блок " + hit.side())
    is Hit.OnEntity -> chat.print("сущность " + hit.entity().name())
    is Hit.None -> {}
}
```

Развёрнутая форма даёт больше контроля:

```kotlin
raycast.crosshair(6.0, true) { it.isPlayer() }   // true — учитывать блоки
```

## Луч не из прицела

Иногда нужно посмотреть не туда, куда смотришь, а туда, куда собираешься:

```kotlin
val aim = rotations.lookAt(target.position())
val hit = raycast.from(aim, 6.0, true) { !it.isAlly() }
```

Так проверяют, не мешает ли блок ударить по цели, до того как поворачивать голову.

## Только блоки

```kotlin
val hit = raycast.blocks(from, to, RaycastShape.COLLIDER, FluidHandling.NONE)
```

| Форма | Что учитывается |
|---|---|
| `COLLIDER` | форма для столкновений |
| `OUTLINE` | контур, как для выделения |
| `VISUAL` | видимая форма |

| Жидкости | Что считается препятствием |
|---|---|
| `NONE` | вода и лава не мешают |
| `SOURCE_ONLY` | только источники |
| `ANY` | любая жидкость |

## Видно ли точку

```kotlin
if (raycast.canSee(player.eyePosition(), target.position())) {
    // между нами нет блоков
}
```

## Куда именно попаду

`hitOn` возвращает точку на коробке сущности, в которую упрётся луч, или `null`, если промах:

```kotlin
val point = raycast.hitOn(target, player.eyePosition(), aimPoint)
```

Готовый способ выбрать хорошую точку удара — [`combat.attackPoint`](../actions/interaction.md).

## Про производительность

Лучи не бесплатны. Один-два на тик — нормально, десятки по всем сущностям каждый тик — уже нет. Сначала отсеивай по расстоянию через `world.entitiesNear(...)`, и только потом стреляй лучом в оставшихся.
