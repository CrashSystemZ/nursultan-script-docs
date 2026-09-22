# Рендер 3D

`Render3D` приходит из `Render3DEvent` и рисует в мировых координатах; позицию камеры, снятую в момент события, из точек вычитают за тебя. Члены самого события — `render()`, `tickDelta()`, `camera()`, `viewMatrix()` и `projectionMatrix()` (API 2) — в [списке событий](../events/reference.md).

```kotlin
on<Render3DEvent> { e ->
    val targets = world.entitiesNear(player.position(), 48.0, filters.attackable())
    for (target in targets) {
        e.render().entityBox(target, Colors.RED, true)
    }
    val nearest = targets.minByOrNull { it.distanceTo(player) } ?: return@on
    e.render().tracer(nearest.renderPosition().add(0.0, nearest.height() / 2.0, 0.0), Colors.RED, true)
}
```

## Что можно нарисовать

| Метод | Тип | Описание |
|---|---|---|
| `line(from, to, argb, throughWalls)` | `void` | одна линия в мировых координатах |
| `box(box, argb, throughWalls)` | `void` | 12 рёбер коробки |
| `filledBox(box, argb, throughWalls)` | `void` | залитые грани коробки |
| `entityBox(entity, argb, throughWalls)` | `void` | коробка по интерполированной позиции, ширине и высоте |
| `tracer(target, argb, throughWalls)` | `void` | линия от центра ближней плоскости камеры к точке |
| `triangle(a, b, c, argb, throughWalls)` | `void` | залитый треугольник |
| `quad(a, b, c, d, argb, throughWalls)` | `void` | залитый четырёхугольник |
| `polygon(points, argb, throughWalls)` | `void` | веер треугольников от первой точки, меньше 3 точек — ничего |

Точки `Vec` и `Box` — мировые координаты в блоках, см. [векторы, коробки, углы](../game/math.md). `argb` упакован как `0xAARRGGBB`; константы и хелперы — на странице [рендер 2D](render-2d.md).

## Сквозь стены

`throughWalls` — последний аргумент каждого вызова выше, он выбирает меш: `true` пишет в меш без теста глубины, и фигура видна сквозь блоки.

## Подписи

Текста в мире нет. Переведи мировую точку в экранные координаты через `render.project(...)` внутри `Render2DEvent` и рисуй подпись там — см. [рендер 2D](render-2d.md).
