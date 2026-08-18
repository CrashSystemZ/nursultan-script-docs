# Повороты

`rotations` — это `client.rotations()`. Применённый поворот записывается в следующий пакет движения, то есть его видит сервер; угол квантуется по шагу чувствительности мыши.

```kotlin
on<PrePlayerTickEvent> {
    val target = combat.target() ?: return@on
    rotations.apply(
        rotations.lookAt(target),
        RotationOptions.DEFAULT.priority(RotationPriority.NOW)
    )
}
```

## Прочитать углы

| Метод | Тип | Описание |
|---|---|---|
| `rotations.player()` | `Rotation` | yaw/pitch игрока в градусах, вместе с подменой |
| `rotations.camera()` | `Rotation` | угол, который держит обработчик, равен `player()` в простое |
| `rotations.lastSent()` | `Rotation` | yaw/pitch из последнего отправленного пакета движения |
| `rotations.angleTo(point)` | `float` | градусы 0..180 от текущего взгляда до `Vec` |

Любой метод бросает `ScriptStateException` без игрока или мира.

## Куда смотреть

| Метод | Тип | Описание |
|---|---|---|
| `rotations.lookAt(point)` | `Rotation` | yaw/pitch от позиции глаз к точке `Vec` |
| `rotations.lookAt(entity)` | `Rotation` | yaw/pitch к точке атаки `NEAREST` этой сущности |

Сам `Rotation` — на странице [Векторы, коробки, углы](../game/math.md#rotation), точки атаки — на странице [Взаимодействие](interaction.md#куда-бить).

## Применить

| Метод | Тип | Описание |
|---|---|---|
| `rotations.apply(rotation)` | `void` | ставит в очередь с `RotationOptions.DEFAULT` (только главный поток) |
| `rotations.apply(rotation, options)` | `void` | ставит в очередь, null-опции означают `DEFAULT` (только главный поток) |

Применяется на ближайшем pre-player-tick и снапится к кванту мыши с дизерингом; один вызов действует один тик.
Ничего не делает, пока поворот держит блокировку обработчика, и бросает `ScriptStateException` без игрока или мира.

## Настройки поворота

| Метод | Тип | Описание |
|---|---|---|
| `RotationOptions.DEFAULT` | `RotationOptions` | статическое поле `(NORMAL, false, false, false, false, FAST)` |
| `options.priority()` | `RotationPriority` | место в очереди поворотов этого тика |
| `options.priority(value)` | `RotationOptions` | копия с новым приоритетом |
| `options.strongCorrection()` | `boolean` | true отключает ремап WASD, движение идёт по подменённому yaw |
| `options.strongCorrection(value)` | `RotationOptions` | копия с новым значением |
| `options.smoothBackRotation()` | `boolean` | true возвращает голову плавно, false — рывком |
| `options.smoothBackRotation(value)` | `RotationOptions` | копия с новым значением |
| `options.backRotation()` | `BackRotation` | форма возврата, читается только при `smoothBackRotation` |
| `options.backRotation(value)` | `RotationOptions` | копия с новым значением |
| `options.clientSide()` | `boolean` | хранимый флаг (устарело, ничего не делает: поворот всегда уходит на сервер) |
| `options.clientSide(value)` | `RotationOptions` | копия с новым значением (устарело, ничего не делает: поворот всегда уходит на сервер) |
| `options.normalizeMouseMovement()` | `boolean` | хранимый флаг (устарело, ничего не делает: угол всегда снапится к шагу мыши) |
| `options.normalizeMouseMovement(value)` | `RotationOptions` | копия с новым значением (устарело, ничего не делает: угол всегда снапится к шагу мыши) |
| `RotationOptions(priority, clientSide, strongCorrection, smoothBackRotation, normalizeMouseMovement, backRotation)` | `RotationOptions` | канонический конструктор (бросает `NullPointerException`, если `priority` или `backRotation` null) |

Запись неизменяемая: каждый сеттер возвращает копию, а `DEFAULT` не меняется.
Четыре устаревших члена собраны на странице [Что больше ничего не делает](../extras/api-versions.md#что-больше-ничего-не-делает).

## Порядок внутри тика

**`RotationPriority`**

| Константа | Описание |
|---|---|
| `NOW` | вес 200, применяется последним, перебивает младшие |
| `NORMAL` | вес 0, значение в `DEFAULT` |
| `LATER` | вес -200, применяется первым |

**`BackRotation`**

| Константа | Описание |
|---|---|
| `FAST` | 50% лерпа за тик к камере, через 20 тиков рывком |
| `SMOOTH` | до 12 градусов за тик по каждой оси, через 26 тиков рывком |

Повороты, поставленные в очередь за один тик, применяются в порядке `LATER` → `NORMAL` → `NOW`, поэтому старший приоритет записывается последним.
