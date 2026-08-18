# Предсказание

`prediction` — это `client.prediction()`. Он прогоняет локального игрока вперёд на теневой сущности, копирующей его атрибуты движения, и считает полёт снаряда от последней отправленной позиции.

```kotlin
on<Render3DEvent> { e ->
    val fall = prediction.path(10) { it.onGround() }   // останавливается на первом тике с землёй
    val landing = fall.last().position()
    e.render().box(Box.around(landing, 0.2), Colors.RED, true)
}
```

## Симуляция движения

| Метод | Тип | Описание |
|---|---|---|
| `prediction.after(ticks)` | `PredictionResult` | состояние через `ticks` симулированных тиков, живой ввод с клавиатуры |
| `prediction.after(ticks, input)` | `PredictionResult` | то же, `input` зажат каждый симулированный тик |
| `prediction.path(ticks)` | `List<PredictionResult>` | по одной неизменяемой записи на тик, нулевой тик не входит |
| `prediction.path(ticks, input)` | `List<PredictionResult>` | то же с фиксированным вводом |
| `prediction.path(ticks, until)` | `List<PredictionResult>` | останавливается на первой записи под `until`, она входит |
| `prediction.path(ticks, input, until)` | `List<PredictionResult>` | фиксированный ввод плюс условие остановки |

`ticks` везде 1..100; движение считается на теневой сущности, копирующей скорость движения, скорость приседа, силу прыжка, гравитацию, высоту шага и эффективность движения.
Всем шести нужен мир и клиентский поток (только клиентский поток); `ticks` вне диапазона бросает `ScriptException`, вызов изнутри условия `path` — `ScriptStateException`.

## Один тик симуляции

| Метод | Тип | Описание |
|---|---|---|
| `result.position()` | [`Vec`](../game/math.md#vec) | предсказанная позиция ног в блоках мира |
| `result.velocity()` | [`Vec`](../game/math.md#vec) | предсказанная скорость в блоках за тик |
| `result.box()` | [`Box`](../game/math.md#box) | предсказанный хитбокс в мировых координатах |
| `result.onGround()` | `boolean` | предсказанный флаг «на земле» |
| `result.inWater()` | `boolean` | предсказанный флаг касания воды |
| `result.sneaking()` | `boolean` | состояние приседа на этом тике |
| `result.sprinting()` | `boolean` | состояние спринта на этом тике |
| `result.jumping()` | `boolean` | ввод прыжка на этом тике |
| `result.fallDistanceBlocks()` | `double` | накопленная высота падения в блоках |

## Ввод

| Метод | Тип | Описание |
|---|---|---|
| `moveInput(forward, backward, left, right, jump, sneak, sprint)` | `MoveInput` | собирает запись ввода, все аргументы по умолчанию `false` |
| `MoveInput.NONE` | `MoveInput` | все семь флагов выключены |
| `input.forward()` | `boolean` | зажата клавиша вперёд |
| `input.backward()` | `boolean` | зажата клавиша назад |
| `input.left()` | `boolean` | зажат стрейф влево |
| `input.right()` | `boolean` | зажат стрейф вправо |
| `input.jump()` | `boolean` | зажат прыжок |
| `input.sneak()` | `boolean` | зажат присед |
| `input.sprint()` | `boolean` | зажат спринт |
| `input.copy(forward, backward, left, right, jump, sneak, sprint)` | `MoveInput` | копия с заменой полей, остальное берётся из приёмника |

`MoveInputEvent.toInput()` отдаёт реально зажатые клавиши — см. [Список событий](../events/reference.md).

## Снаряды

| Метод | Тип | Описание |
|---|---|---|
| `prediction.projectile(kind)` | `ProjectilePath` | траектория по последнему отправленному повороту, штатная сила вида |
| `prediction.projectile(kind, aim)` | `ProjectilePath` | `aim` = null — последний отправленный поворот, штатная сила вида |
| `prediction.projectile(kind, aim, power)` | `ProjectilePath` | своя сила запуска |
| `path.points()` | `List<Vec>` | точки траектории, по одной на тик, от точки запуска |
| `path.lands()` | `boolean` | найдено попадание в блок или сущность |
| `path.landingPosition()` | `Vec?` | точка попадания, null если не попал |
| `path.hitEntity()` | [`Entity?`](../game/entities.md) | задетая сущность, иначе null |
| `path.hitsBlock()` | `boolean` | попадание пришлось в блок |
| `path.flightTicks()` | `int` | `points().size() - 1`, минимум 0 |

Начало — последняя отправленная позиция плюс высота глаз, в скорость запуска входит текущее движение игрока, а `aim` — это [`Rotation`](../game/math.md#rotation).
Всем трём нужен мир и клиентский поток (только клиентский поток); `kind` = null или `power` не больше 0 бросает `ScriptException`.

| Константа | Описание |
|---|---|
| `ARROW` | физика стрелы, штатная сила 3.0 |
| `TRIDENT` | физика трезубца, штатная сила 2.5 |
| `ENDER_PEARL` | физика жемчуга Края, штатная сила 1.5 |
| `SPLASH_POTION` | физика взрывного зелья, сила 0.5, наклон прицела -20° |
| `SNOWBALL` | физика снежка, штатная сила 1.5 |
| `WIND_CHARGE` | физика заряда ветра, штатная сила 1.5 |
