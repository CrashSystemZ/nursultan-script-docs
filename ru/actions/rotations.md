# Повороты

`rotations` крутит голову игрока. Обычно это нужно, чтобы удар или установка блока улетели туда, куда надо, а не туда, куда смотрит игрок.

## Прочитать текущие углы

```kotlin
rotations.player()      // куда смотрит игрок
rotations.camera()      // куда смотрит камера
rotations.lastSent()    // что последним ушло на сервер
```

Разница между `player()` и `lastSent()` важна: скрипт может слать серверу один угол, а на экране голова останется где была.

## Куда смотреть

```kotlin
val aim = rotations.lookAt(target.position())
val aim = rotations.lookAt(target)            // сразу по сущности
```

`Rotation` — это пара углов, `yawDegrees` и `pitchDegrees`:

```kotlin
aim.yawDegrees()
aim.pitchDegrees()
aim.direction()                     // Vec, куда смотрит
aim.angleTo(other)                  // на сколько градусов надо довернуть
aim.yawDeltaTo(other)
aim.pitchDeltaTo(other)
```

`angleTo` удобен, чтобы не бить, когда цель ещё далеко от прицела:

```kotlin
if (rotations.player().angleTo(aim) > 30f) return@on
```

## Применить

```kotlin
rotations.apply(aim)
```

Один вызов действует на текущий тик. Хочешь держать поворот — вызывай каждый тик:

```kotlin
on<PrePlayerTickEvent> {
    val target = combat.target() ?: return@on
    rotations.apply(rotations.lookAt(combat.attackPoint(target)))
}
```

`rotations.angleTo(point)` — короткий способ узнать, на сколько градусов цель в стороне, не собирая `Rotation` руками.

## Настройки поворота

Если нужен контроль над тем, как именно поворачивается голова:

```kotlin
rotations.apply(
    aim,
    RotationOptions.DEFAULT
        .priority(RotationPriority.NOW)
        .clientSide(false)
        .backRotation(BackRotation.SMOOTH)
)
```

| Параметр | Что делает |
|---|---|
| `priority` | кто победит, если поворот просят несколько модулей: `NOW`, `NORMAL`, `LATER` |
| `clientSide` | поворачивать только на экране, серверу не слать |
| `strongCorrection` | сильнее подтягивать движение под новый угол |
| `smoothBackRotation` | плавно возвращать голову обратно |
| `normalizeMouseMovement` | подгонять угол под шаг мыши |
| `backRotation` | как возвращаться: `FAST` или `SMOOTH` |

Все методы возвращают новый объект, так что их можно писать цепочкой, а `RotationOptions.DEFAULT` не меняется.

## Кто победит

Повороты просят и встроенные модули, и скрипты. Побеждает больший приоритет; при равных — тот, кто попросил позже в этом тике. Поэтому `NOW` бери только когда действительно нужно перебить всех остальных.
