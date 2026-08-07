# Рендер 3D

Событие `Render3DEvent` вызывается каждый кадр и рисует прямо в мире. Оно несёт `render()`, `tickDelta()` и позицию камеры `camera()`.

Координаты здесь — мировые, те же, что у `player.position()`.

```kotlin
on<Render3DEvent> { e ->
    val r = e.render()
    for (target in world.players()) {
        if (target.isSelf() || target.isAlly()) continue
        r.entityBox(target, Colors.RED, true)
    }
}
```

## Что можно нарисовать

| Метод | Что рисует |
|---|---|
| `box(box, argb, throughWalls)` | рёбра коробки |
| `filledBox(box, argb, throughWalls)` | залитую коробку |
| `entityBox(entity, argb, throughWalls)` | коробку вокруг сущности |
| `line(from, to, argb, throughWalls)` | линию между двумя точками |
| `tracer(target, argb, throughWalls)` | линию от прицела к точке |
| `triangle(a, b, c, argb, throughWalls)` | залитый треугольник |
| `quad(a, b, c, d, argb, throughWalls)` | залитый четырёхугольник |
| `polygon(points, argb, throughWalls)` | залитый выпуклый многоугольник |

Точки — это мировые координаты, те же, что у `player.position()`. Полупрозрачный круг на земле вокруг тебя:

```kotlin
on<Render3DEvent> { e ->
    val centre = player.renderPosition().add(0.0, 0.02, 0.0)
    val ring = (0 until 24).map {
        val angle = Math.toRadians(it * 15.0)
        centre.add(Math.cos(angle) * 3.0, 0.0, Math.sin(angle) * 3.0)
    }
    e.render().polygon(ring, Colors.rgba(90, 200, 255, 60), true)
}
```

`polygon` разворачивает точки веером от первой, поэтому **выпуклую** фигуру рисует правильно, а вогнутую — нет. Точки должны идти по порядку вдоль контура, и лежать в одной плоскости, если хочется плоскую фигуру.

Последний аргумент везде один и тот же: `true` — видно сквозь блоки, `false` — прячется за ними.

## Коробки

Коробку можно взять готовую или собрать:

```kotlin
entity.box()                              // коробка сущности
world.block(x, y, z).box()                // коробка блока
Box.around(position, 0.5)                 // кубик вокруг точки

entity.box().expand(0.1)                  // чуть больше
```

## Трейсеры

Линия от центра экрана к цели — самый заметный способ показать, где враг:

```kotlin
on<Render3DEvent> { e ->
    val enemy = world.nearestEntity(player.position(), 64.0, filters.attackable()) ?: return@on
    e.render().tracer(enemy.renderPosition().add(0.0, enemy.height() / 2.0, 0.0), Colors.RED, true)
}
```

## Плавность

Для рисования бери `renderPosition()`, а не `position()`: она уже сглажена между тиками, и коробка не будет дёргаться.

```kotlin
val box = entity.box().offset(entity.renderPosition().subtract(entity.position()))
```

Или проще — `entityBox`, он это делает сам.

## Подписи над сущностями

Текст в мире не рисуется: он живёт на HUD. Чтобы подписать сущность, переведи её позицию в экранные координаты через [`project`](render-2d.md) в `Render2DEvent`.

## Не перегружай кадр

Рисование дешевле, чем расчёты, но не бесплатно. Сотни коробок каждый кадр заметно просадят fps. Отбирай сущности заранее — по расстоянию и фильтрам — и держи список в переменной, обновляя его в `ClientTickEvent`.

```kotlin
var targets: List<Entity> = emptyList()

on<ClientTickEvent> {
    whenInGame {
        targets = world.entitiesNear(player.position(), 48.0, filters.attackable())
    }
}

on<Render3DEvent> { e ->
    for (target in targets) {
        e.render().entityBox(target, Colors.RED, true)
    }
}
```
