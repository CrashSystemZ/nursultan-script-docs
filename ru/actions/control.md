# Движение

`control` — это `game.control()`, три прямые записи в локального игрока. `gameSettings` — это `game.settings()`, чтение и запись куска ванильных настроек.

```kotlin
on<ClientTickEvent> {
    control.sprinting(true)
}

on<MoveInputEvent> {
    it.forward(true)   // держать вперёд в этом тике
    it.jump(false)
}
```

## Прямые команды

| Метод | Тип | Описание |
|---|---|---|
| `control.sprinting(value)` | `void` | ставит флаг бега локального игрока (только главный поток) |
| `control.sneaking(value)` | `void` | ставит флаг приседа локального игрока (только главный поток) |
| `control.jump()` | `void` | прыгает, вне земли ничего не делает (только главный поток) |

Любой метод бросает [`ScriptThreadException`](../extras/limits.md#исключения) вне клиентского потока и `ScriptStateException`, когда мир не загружен.

## Подмена ввода

Семь сырых флагов движения на тик — это [`MoveInputEvent`](../events/reference.md#moveinputevent); после обработчика игра собирает ввод игрока заново из них. Движение мыши за кадр — [`LookInputEvent`](../events/reference.md#lookinputevent).

Что зажато физически прямо сейчас, независимо от ввода тика, — [Клавиши и бинды](keys.md); наведение головы в точку — [Повороты](rotations.md).

## Настройки игры

| Метод | Тип | Описание |
|---|---|---|
| `gameSettings.perspective()` | `Perspective` | выбранный игроком вид, `FIRST_PERSON` если настройки недоступны (API 2) |
| `gameSettings.perspective(value)` | `void` | ставит вид, null ничего не делает (API 2) (только главный поток) |
| `gameSettings.scaleFactor()` | `double` | масштаб интерфейса окна, например 1.0 / 2.0 / 3.0 (API 2) |
| `gameSettings.mouseSensitivity()` | `double` | чувствительность мыши 0..1, 0.5 если настройки недоступны (API 2) |
| `gameSettings.mouseSensitivity(value)` | `void` | ставит чувствительность, обрезается до 0..1 (API 2) (только главный поток) |

Ни одна из настроек не возвращается назад при выключении скрипта; `scaleFactor()` — то число, на которое делят пиксели [рендера 2D](../ui/render-2d.md), чтобы получить координаты ванильного интерфейса.
`PerspectiveEvent` приходит на каждую смену вида, кто бы её ни сделал — [Список событий](../events/reference.md#perspectiveevent).

### Perspective

| Метод | Тип | Описание |
|---|---|---|
| `perspective.firstPerson()` | `boolean` | true только для `FIRST_PERSON` (API 2) |
| `perspective.thirdPerson()` | `boolean` | отрицание `firstPerson()` (API 2) |
| `perspective.frontView()` | `boolean` | true только для `THIRD_PERSON_FRONT` (API 2) |

### Константы Perspective

| Константа | Описание |
|---|---|
| `FIRST_PERSON` | камера в голове игрока |
| `THIRD_PERSON_BACK` | камера позади игрока |
| `THIRD_PERSON_FRONT` | камера перед игроком, лицом к нему |

## Замедление

Множитель движения, который игра применяет во время использования предмета, — [`SlowdownEvent`](../events/reference.md#slowdownevent); записанное значение подставляется вместо ванильного и не ограничивается.
