# Виды настроек

Настройки объявляются на верхнем уровне скрипта и появляются под его карточкой во вкладке Scripts. Порядок в меню — порядок объявления.

## Список

| Настройка | Как создать | Что читается через `by` |
|---|---|---|
| Чекбокс | `checkBox("Имя", true)` | `Boolean` |
| Слайдер | `slider("Имя", 5f, 1f, 20f, step = 1f)` | `Float` |
| Слайдер-диапазон | `rangeSlider("Имя", 2f, 8f, 0f, 20f)` | `ClosedFloatingPointRange<Float>` |
| Поле ввода | `input("Имя", "текст")` | `String` |
| Один вариант | `selectable("Имя", "A", "B", selected = "A")` | `String` |
| Несколько вариантов | `combo("Имя", "A", "B", selected = listOf("A"))` | `List<String>` |
| Цвет | `colorPicker("Имя", Colors.rgba(90, 200, 255, 255))` | `Int` |
| Хоткей | `hotkey("Имя", Key.G) { ... }` | `Key` |
| Кнопка | `button("Имя") { ... }` | — |

У слайдера есть целочисленная форма: `slider("Имя", 5, 1, 20)` принимает `Int` и `intValue()` вернёт `Int` без округления руками.

## Чтение и запись

У каждой настройки есть `value()` для чтения и `value(...)` для записи:

```kotlin
val speed = slider("Скорость", 5f, 1f, 20f)

speed.value()          // 5.0
speed.intValue()       // 5
speed.value(12f)
speed.min()            // 1.0
speed.max()            // 20.0
```

`rangeSlider` устроен чуть иначе — у него две границы:

```kotlin
val delay = rangeSlider("Задержка", 2f, 5f, 0f, 20f)

delay.from()           // 2.0
delay.to()             // 5.0
delay.value(3f, 8f)
```

## Единицы у слайдеров

Слайдер рисует голое число. `postfix(...)` дописывает к нему единицу — те же, что у обычных слайдеров клиента:

```kotlin
val delay by slider("Задержка", 200f, 0f, 1000f, 50f).postfix(Postfixes.MS)
val range by rangeSlider("Дистанция", 3f, 5f, 0f, 6f, 0.1f).postfix(Postfixes.BLOCKS)
```

Набор фиксированный — ты выбираешь единицу, а не пишешь текст сам:

| Значение | Рисуется как |
|---|---|
| `Postfixes.PERCENT` | `%` |
| `Postfixes.DEGREES` | `°` |
| `Postfixes.SECONDS` | `s` |
| `Postfixes.TICKS` | `t` |
| `Postfixes.BLOCKS` | `b` |
| `Postfixes.MS` | `ms` |
| `Postfixes.MULTIPLIER` | `x` |
| `Postfixes.HEALTH` | `hp` |

`postfix(...)` возвращает сам слайдер, поэтому его можно дописать прямо перед `by`. Без него число останется голым.

`hotkey` создаётся вместе с действием — оно выполнится при нажатии, пока скрипт включён:

```kotlin
hotkey("Паника", Key.RIGHT_SHIFT) { enabled = false }
```

## Реакция на изменение

```kotlin
val color = colorPicker("Цвет", Colors.CYAN)

color.onChange { argb ->
    chat.print("новый цвет: $argb")
}
```

У `rangeSlider` в обработчик приходят обе границы, у `combo` — список выбранных [entry](entries.md).

## Вложенность и видимость

Любую настройку можно положить внутрь чекбокса — она будет отображаться под ним как подпункт:

```kotlin
val fancy = checkBox("Красивости", false)
val alpha = slider(fancy, "Прозрачность", 0.5f, 0f, 1f, 0.05f)
val glow = checkBox(fancy, "Свечение", true)
```

И спрятать по условию:

```kotlin
alpha.visibleWhen { fancy.value() }
```

Условие проверяется при отрисовке меню, так что писать в нём тяжёлые вычисления не надо.

## Цвета

Цвет — это обычный `Int` в формате `0xAARRGGBB`. Собирать его руками не обязательно:

```kotlin
Colors.rgb(90, 200, 255)
Colors.rgba(90, 200, 255, 128)
Colors.WHITE
Colors.withAlpha(Colors.RED, 100)
Colors.fade(Colors.RED, 0.5f)          // приглушить по альфе
Colors.mix(Colors.RED, Colors.BLUE, 0.5f)
Colors.red(argb)                        // разобрать обратно
```
