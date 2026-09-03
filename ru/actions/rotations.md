# Повороты

`rotations` — это `client.rotations()`. Применённый поворот записывается в следующий пакет движения как есть, то есть его видит сервер; клиент больше не снапит его к шагу мыши за тебя, а угол, который реально уйдёт, даёт `rotations.quantized(rotation)`.

```kotlin
on<PrePlayerTickEvent> {
    val target = combat.target() ?: return@on
    val aim = rotations.quantized(rotations.lookAt(target))
    rotations.apply(aim, RotationOptions.DEFAULT.priority(RotationPriority.NOW))
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
| `rotations.quantized(rotation)` | `Rotation` | копия, снапнутая к сетке чувствительности мыши (API 6) |

`quantized` кладёт на шаг мыши обе оси, разворачивает yaw в систему координат игрока и зажимает pitch в -90..90 градусов.
Рейкаст по квантованному значению проверяет ровно тот угол, который ты применишь.

Сам `Rotation` — на странице [Векторы, коробки, углы](../game/math.md#rotation), точки атаки — на странице [Взаимодействие](interaction.md#куда-бить).

## Применить

| Метод | Тип | Описание |
|---|---|---|
| `rotations.apply(rotation)` | `void` | ставит в очередь с `RotationOptions.DEFAULT` (только главный поток) |
| `rotations.apply(rotation, options)` | `void` | ставит в очередь, null-опции означают `DEFAULT` (только главный поток) |
| `rotations.locked()` | `boolean` | хендлер занят другим поворотом, и `apply` сейчас ничего не делает |

Применяется на ближайшем pre-player-tick ровно в том виде, в каком ты его передал; один вызов действует один тик.
Ничего не делает, пока поворот держит блокировку обработчика, и бросает `ScriptStateException` без игрока или мира.

## Настройки поворота

| Метод | Тип | Описание |
|---|---|---|
| `RotationOptions.DEFAULT` | `RotationOptions` | статическое поле `(NORMAL, false, BackRotations.SNAP)` |
| `options.priority()` | `RotationPriority` | место в очереди поворотов этого тика |
| `options.priority(value)` | `RotationOptions` | копия с новым приоритетом |
| `options.strongCorrection()` | `boolean` | true отключает ремап WASD, движение идёт по подменённому yaw |
| `options.strongCorrection(value)` | `RotationOptions` | копия с новым значением |
| `options.backRotation()` | `BackRotation` | форма возврата головы к камере |
| `options.backRotation(value)` | `RotationOptions` | копия с новым значением |
| `options.smoothBackRotation()` | `boolean` | хранимый флаг (устарело, используй `backRotation`) |
| `options.smoothBackRotation(value)` | `RotationOptions` | копия с новым значением, true превращает возврат `SNAP` в `HUMANIZED` (устарело, используй `backRotation`) |
| `options.clientSide()` | `boolean` | хранимый флаг (устарело) (ничего не делает: поворот всегда уходит на сервер) |
| `options.clientSide(value)` | `RotationOptions` | копия с новым значением (устарело) (ничего не делает: поворот всегда уходит на сервер) |
| `options.normalizeMouseMovement()` | `boolean` | хранимый флаг (устарело, используй `rotations.quantized`) (ничего не делает: значение нигде не читается) |
| `options.normalizeMouseMovement(value)` | `RotationOptions` | копия с новым значением (устарело, используй `rotations.quantized`) (ничего не делает: значение нигде не читается) |
| `RotationOptions(priority, clientSide, strongCorrection, smoothBackRotation, normalizeMouseMovement, backRotation)` | `RotationOptions` | канонический конструктор (бросает `NullPointerException`, если `priority` или `backRotation` null) |
| `RotationOptions(priority, strongCorrection, backRotation)` | `RotationOptions` | конструктор без устаревших флагов (API 6) |

Запись неизменяемая: каждый сеттер возвращает копию, а `DEFAULT` не меняется.
Устаревшие члены собраны на странице [Что больше ничего не делает](../extras/api-versions.md#что-больше-ничего-не-делает).

## Порядок внутри тика

### RotationPriority

| Константа | Описание |
|---|---|
| `NOW` | вес 200, применяется последним, перебивает младшие |
| `NORMAL` | вес 0, значение в `DEFAULT` |
| `LATER` | вес -200, применяется первым |

Повороты, поставленные в очередь за один тик, применяются в порядке `LATER` → `NORMAL` → `NOW`, поэтому старший приоритет записывается последним.

## Возврат головы

| Метод | Тип | Описание |
|---|---|---|
| `BackRotations.SNAP` | `BackRotation` | 1 тик, возвращает голову сразу, значение в `DEFAULT` (API 6) |
| `BackRotations.INSTANT` | `BackRotation` | 2 тика, половина дельты и следом остаток (API 6) |
| `BackRotations.HUMANIZED` | `BackRotation` | 20 тиков по записанным человеческим дельтам (API 6) |
| `BackRotation.FAST` | `BackRotation` | тот же возврат, что у `BackRotations.SNAP` (устарело, используй `BackRotations.SNAP`) |
| `BackRotation.SMOOTH` | `BackRotation` | тот же возврат, что у `BackRotations.HUMANIZED` (устарело, используй `BackRotations.HUMANIZED`) |

Когда поворот больше никто не применяет, обработчик уводит подменённую голову обратно к камере той `BackRotation`, что стояла в последних применённых опциях.

### Своя форма возврата

| Метод | Тип | Описание |
|---|---|---|
| `BackRotation.step(from, to, tick)` | `Rotation?` | угол на этот тик, null — возврат закончен (API 6) |
| `BackRotation.maxTicks()` | `int` | сколько тиков даётся возврату, по умолчанию 20 (API 6) |
| `backRotation(maxTicks) { from, to, tick -> }` | `BackRotation` | форма интерфейса из DSL, `maxTicks` по умолчанию 20 (API 6) |

`from` — куда смотрит подменённая голова, `to` — куда смотрит настоящая камера, `tick` считается с нуля.
Твои шаги применяются как есть, поэтому квантуй их сам; обработчик завершает возврат, когда доходит до `maxTicks`.
