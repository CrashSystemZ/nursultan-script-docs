# Движение

Двигать игрока можно двумя способами: командой прямо сейчас или подменой ввода на этот тик.

## Прямые команды

```kotlin
control.jump()
control.sprinting(true)
control.sneaking(true)
```

`jump()` прыгает сразу, если игрок стоит на земле. `sprinting` и `sneaking` включают и выключают состояние.

Годится, когда действие разовое: подпрыгнуть при отбрасывании, присесть на секунду.

## Подмена ввода

`MoveInputEvent` — это то, что игра собралась сделать в этом тике. Все поля можно переписать:

```kotlin
on<MoveInputEvent> { e ->
    e.forward(true)
    e.sprint(true)
    e.jump(false)
}
```

| Поле | Что значит |
|---|---|
| `forward()`, `backward()` | вперёд и назад |
| `left()`, `right()` | вбок |
| `jump()` | прыжок |
| `sneak()` | присед |
| `sprint()` | бег |

Годится, когда нужно держать состояние: не давать бежать, вести игрока вперёд, блокировать прыжок.

## Приоритет

На один и тот же ввод претендуют и модули клиента, и скрипты. Хочешь сказать последнее слово — подписывайся с `Priority.LAST`:

```kotlin
on<MoveInputEvent>(priority = Priority.LAST) {
    it.sprint(false)
}
```

Пример: погасить спринт ровно на один тик, чтобы удар прошёл как критический.

```kotlin
var blockSprintTicks = 0

on<MoveInputEvent>(priority = Priority.LAST) {
    if (blockSprintTicks > 0) {
        blockSprintTicks--
        it.sprint(false)
    }
}

// где-то в логике боя
blockSprintTicks = 1
```

## Мышь

`LookInputEvent` даёт движение мыши за кадр, и его тоже можно переписать:

```kotlin
on<LookInputEvent> { e ->
    e.deltaX(e.deltaX() * 0.5)     // вдвое медленнее по горизонтали
    e.deltaY(e.deltaY() * 0.5)
}
```

Отменишь событие — мышь вообще перестанет крутить камеру.

Поворачивать голову по цели — это не сюда, это [Повороты](rotations.md).

## Настройки игры

`gameSettings` — это настройки самого Minecraft, то, что игрок выставил в меню игры:

```kotlin
gameSettings.perspective()                               // FIRST_PERSON | THIRD_PERSON_BACK | THIRD_PERSON_FRONT
gameSettings.perspective(Perspective.THIRD_PERSON_BACK)  // ровно то же, что нажатие F5

gameSettings.scaleFactor()                               // масштаб интерфейса числом: 1.0, 2.0, 3.0 …
gameSettings.mouseSensitivity()                          // 0..1, где 0.5 — те самые «100%» из настроек
gameSettings.mouseSensitivity(0.5)
```

Сама `Perspective` отвечает на три вопроса о себе: `firstPerson()`, `thirdPerson()`, `frontView()`.

Писать любую из этих настроек можно только с клиентского потока: внутри обработчика ты уже на нём, откуда угодно ещё — оберни в `onClientThread { }`. Ничего не возвращается обратно, когда скрипт выключают, так что верни сам:

```kotlin
var previous = gameSettings.perspective()

onEnable {
    previous = gameSettings.perspective()
    gameSettings.perspective(Perspective.THIRD_PERSON_BACK)
}

onDisable { gameSettings.perspective(previous) }
```

`PerspectiveEvent` приходит при каждой смене вида, кто бы её ни сделал — игрок, твой скрипт или чужой:

```kotlin
on<PerspectiveEvent> { e ->
    log.info("${e.previous()} -> ${e.current()}")
}
```

Читаешь ты всегда выбор самого игрока. Модули клиента, которые навязывают вид, — FreeCam, FreeLook — меняют то, как рисуется кадр, а не опцию, так что `gameSettings.perspective()` по-прежнему показывает, что выбрал игрок.

`scaleFactor()` — это мостик между [рендером 2D](../ui/render-2d.md), который работает в настоящих пикселях фреймбуфера, и ванильным интерфейсом, который есть те же пиксели, делённые на это число.

## Спросить клавишу напрямую

События говорят, когда клавиша *изменилась*. `keys` говорит, что зажато **прямо сейчас**, — а это то, что нужно HUD'у:

```kotlin
on<Render2DEvent> { e ->
    if (keys.isDown(Key.LEFT_SHIFT)) {
        e.render().text("крадусь", 10f, 10f, 10f, Colors.WHITE)
    }
}
```

```kotlin
keys.isDown(Key.G)
keys.mouseDown(0)          // 0 левая, 1 правая, 2 средняя
keys.mouseX()  keys.mouseY()   // курсор, в тех же пикселях, в которых рисует Render2DEvent
keys.cursorLocked()        // false, пока открыт экран
keys.lockCursor()
keys.unlockCursor()
keys.inputBlocked()        // игрок печатает в чат или в текстовое поле
```

Проверяй `inputBlocked()` до того, как реагировать на буквенные клавиши, иначе скрипт сработает, пока игрок пишет сообщение.

Это чтение железа, поэтому вернётся `true`, даже когда открыт экран и сама игра клавишу игнорирует. `MoveInputEvent` — противоположность: это то, что игра решила делать в этом тике, уже отфильтрованное.

## Замедление

`SlowdownEvent` вызывается, когда игра собирается замедлить движение: еда, щит, натянутый лук. Множитель можно переписать:

```kotlin
on<SlowdownEvent> { e ->
    e.multiplier(1f)     // не замедляться вообще
}
```
