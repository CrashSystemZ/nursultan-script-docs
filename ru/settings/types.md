# Виды настроек

Настройка создаётся на верхнем уровне скрипта и попадает под его карточку во вкладке Scripts в порядке создания. Делегат `by` читает и пишет живое значение.

```kotlin
val enabled by checkBox("Включено", true)
val delay by slider("Задержка", 200f, 0f, 1000f, 50f).postfix(Postfixes.MS)
val mode by selectable("Режим", "Fast", "Smooth", selected = "Fast")
val color by colorPicker("Цвет", 0xFF5AC8FFL)

on<ClientTickEvent> {
    if (enabled && mode == "Fast") chat.print("$delay мс, $color")
}
```

## Создание

| Метод | Тип | Описание |
|---|---|---|
| `checkBox(name, value)` | `CheckBox` | булева настройка, `value` по умолчанию `false` |
| `checkBox(parent, name, value)` | `CheckBox` | вложена в `parent` (бросает ScriptException, если родитель не чекбокс этого скрипта) |
| `slider(name, value, min, max, step)` | `Slider` | float-слайдер, `step` по умолчанию `1f` (бросает ScriptException при min >= max, step <= 0 или value вне min..max) |
| `slider(parent, name, value, min, max, step)` | `Slider` | вложенный float-слайдер, та же проверка |
| `slider(name, value: Int, min: Int, max: Int, step: Int)` | `Slider` | целочисленная форма, хранится как float |
| `slider(parent, name, value: Int, min: Int, max: Int, step: Int)` | `Slider` | вложенная целочисленная форма |
| `rangeSlider(name, from, to, min, max, step)` | `RangeSlider` | слайдер с двумя ручками (бросает ScriptException, если from..to не влезает в min..max) |
| `rangeSlider(parent, name, from, to, min, max, step)` | `RangeSlider` | вложенный слайдер с двумя ручками, та же проверка |
| `input(name, value, placeholder)` | `Input` | поле ввода; `placeholder` — буквальный текст, виден пока пусто (API 2) |
| `input(parent, name, value, placeholder)` | `Input` | вложенное поле ввода (API 2) |
| `selectable(name, vararg entries: Entry)` | `Selectable` | один из N по объектам `entry()`, первый выбирается сам, если не выбран никто |
| `selectable(parent, name, vararg entries: Entry)` | `Selectable` | вложенный выбор одного по entry |
| `selectable(name, vararg options: String, selected)` | `Selectable` | один из N по строкам, `selected` по умолчанию первый вариант |
| `selectable(parent, name, vararg options: String, selected)` | `Selectable` | вложенный выбор одного по строкам |
| `combo(name, vararg entries: Entry)` | `Combo` | любое число из N по объектам `entry()` |
| `combo(parent, name, vararg entries: Entry)` | `Combo` | вложенный множественный выбор по entry |
| `combo(name, vararg options: String, selected)` | `Combo` | любое число из N по строкам, `selected` по умолчанию пуст |
| `combo(parent, name, vararg options: String, selected)` | `Combo` | вложенный множественный выбор по строкам |
| `colorPicker(name, argb: Int)` | `ColorPicker` | ARGB-цвет, `0xAARRGGBB` |
| `colorPicker(name, argb: Long)` | `ColorPicker` | то же, `Long` сужается — литерал `0xFFRRGGBB` компилируется |
| `colorPicker(parent, name, argb: Int)` | `ColorPicker` | вложенный ARGB-цвет |
| `colorPicker(parent, name, argb: Long)` | `ColorPicker` | вложенный, `Long` сужается до int |
| `hotkey(name, key, action)` | `Hotkey` | клавиша; `action` срабатывает по нажатию, пока скрипт включён |
| `hotkey(parent, name, key, action)` | `Hotkey` | вложенная клавиша |
| `button(name, action)` | `Button` | кнопка в меню; исключения внутри `action` логируются, а не пробрасываются |
| `button(parent, name, action)` | `Button` | вложенная кнопка |
| `entry(name, selected, body)` | `Entry` | один вариант для `selectable` или `combo`, см. [Entry](entries.md) |

Две настройки с одинаковым именем под одним родителем бросают `IllegalArgumentException`; любой создатель бросает `ScriptStateException` после выгрузки скрипта.
`hotkey` принимает `Key` из [Клавиши и бинды](../actions/keys.md).

## Делегат `by`

| Метод | Тип | Описание |
|---|---|---|
| `val x by checkBox(...)` | `Boolean` | читает `value()` |
| `var x by checkBox(...)` | `Boolean` | пишет `value(v)` |
| `val x by slider(...)` | `Float` | читает `value()` |
| `var x by slider(...)` | `Float` | пишет `value(v)`, зажимается в min..max |
| `val x by rangeSlider(...)` | `ClosedFloatingPointRange<Float>` | читает `from()..to()` |
| `var x by rangeSlider(...)` | `ClosedFloatingPointRange<Float>` | пишет обе границы, каждая зажимается в min..max |
| `val x by input(...)` | `String` | читает `value()` |
| `var x by input(...)` | `String` | пишет `value(v)`, null становится `""` |
| `val x by colorPicker(...)` | `Int` | читает ARGB-int, `0xAARRGGBB` |
| `var x by colorPicker(...)` | `Int` | пишет ARGB-int |
| `val x by selectable(...)` | `String` | читает `name()` выбранного entry |
| `var x by selectable(...)` | `String` | выбирает по имени или id (бросает ScriptException, если неизвестно) |
| `val x by combo(...)` | `List<String>` | читает имена выбранных entry, порядок объявления |
| `var x by combo(...)` | `List<String>` | выбирает ровно эти имена (бросает ScriptException, если неизвестно) |
| `val x by hotkey(...)` | `Key` | читает привязанную клавишу |
| `var x by hotkey(...)` | `Key` | перепривязывает клавишу |

У `Button` нет значения и нет делегата.

## Общее у всех настроек

| Метод | Тип | Описание |
|---|---|---|
| `setting.name()` | `String` | отображаемое имя |
| `setting.id()` | `String` | id для поиска; то, что задано через `id(...)`, иначе отображаемое имя |
| `setting.id(stableId)` | `Setting` | задаёт id для поиска (только настройки скрипта) (бросает ScriptException при null или пустой строке) |
| `setting.visible()` | `boolean` | последний посчитанный флаг видимости |
| `setting.visibleWhen(condition)` | `Setting` | ставит предикат видимости `BooleanSupplier` (только настройки скрипта) |

`visible()` пересчитывается при каждой смене значения и при каждой отрисовке меню, то есть `visibleWhen` вызывается с частотой кадров.
У настройки модуля клиента `name()` отдаёт ключ локализации после `.setting.`, `id()` — полный ключ, а любая запись уходит на клиентский поток.

## Чекбокс

| Метод | Тип | Описание |
|---|---|---|
| `checkBox.value()` | `boolean` | текущее состояние галки |
| `checkBox.value(v)` | `CheckBox` | пишет состояние, дёргает слушателей |
| `checkBox.toggle()` | `CheckBox` | пишет инвертированное состояние |
| `checkBox.onChange(listener)` | `CheckBox` | `Consumer<Boolean>` на каждую запись (только настройки скрипта) |
| `checkBox.id(stableId)` | `CheckBox` | ковариантный `Setting.id` |
| `checkBox.visibleWhen(condition)` | `CheckBox` | ковариантный `Setting.visibleWhen` |

`CheckBox` — единственная настройка, которую можно передать как `parent` вложенной настройке.

## Слайдеры

| Метод | Тип | Описание |
|---|---|---|
| `slider.value()` | `float` | текущее значение, в пределах min..max |
| `slider.intValue()` | `int` | текущее значение, округлённое через `Math.round` |
| `slider.value(v)` | `Slider` | пишет значение, зажимая в min..max, без привязки к шагу |
| `slider.min()` | `float` | нижняя граница, заданная при создании |
| `slider.max()` | `float` | верхняя граница, заданная при создании |
| `slider.step()` | `float` | шаг перетаскивания в меню, задан при создании |
| `slider.postfix(unit)` | `Slider` | единица после числа, null убирает её (только настройки скрипта) |
| `slider.onChange(listener)` | `Slider` | `Consumer<Float>` на каждую запись (только настройки скрипта) |
| `slider.id(stableId)` | `Slider` | ковариантный `Setting.id` |
| `slider.visibleWhen(condition)` | `Slider` | ковариантный `Setting.visibleWhen` |
| `rangeSlider.from()` | `float` | нижняя граница выбранного диапазона |
| `rangeSlider.to()` | `float` | верхняя граница выбранного диапазона |
| `rangeSlider.value(from, to)` | `RangeSlider` | пишет обе границы, зажимая в min..max (бросает IllegalArgumentException при from > to) |
| `rangeSlider.min()` | `float` | нижняя граница, заданная при создании |
| `rangeSlider.max()` | `float` | верхняя граница, заданная при создании |
| `rangeSlider.step()` | `float` | шаг перетаскивания в меню, задан при создании |
| `rangeSlider.postfix(unit)` | `RangeSlider` | единица после чисел, null убирает её (только настройки скрипта) |
| `rangeSlider.onChange(listener)` | `RangeSlider` | `BiConsumer<Float, Float>` с from и to (только настройки скрипта) |
| `rangeSlider.id(stableId)` | `RangeSlider` | ковариантный `Setting.id` |
| `rangeSlider.visibleWhen(condition)` | `RangeSlider` | ковариантный `Setting.visibleWhen` |

`step()` управляет только перетаскиванием в меню; `value(...)` к нему никогда не привязывается.

## Текст и цвет

| Метод | Тип | Описание |
|---|---|---|
| `input.value()` | `String` | текущий текст; поле скрипта принимает любой текст, включая переносы |
| `input.value(v)` | `Input` | пишет текст, null становится `""` |
| `input.onChange(listener)` | `Input` | `Consumer<String>` на каждую запись (только настройки скрипта) |
| `input.id(stableId)` | `Input` | ковариантный `Setting.id` |
| `input.visibleWhen(condition)` | `Input` | ковариантный `Setting.visibleWhen` |
| `colorPicker.value()` | `int` | цвет, упакованный как `0xAARRGGBB` |
| `colorPicker.value(argb)` | `ColorPicker` | пишет цвет, без проверок диапазона |
| `colorPicker.onChange(listener)` | `ColorPicker` | `Consumer<Integer>` на каждую запись (только настройки скрипта) |
| `colorPicker.id(stableId)` | `ColorPicker` | ковариантный `Setting.id` |
| `colorPicker.visibleWhen(condition)` | `ColorPicker` | ковариантный `Setting.visibleWhen` |

У поля ввода модуля клиента отклонённый текст откатывается к значению клиента по умолчанию.
`Colors.rgb`, `Colors.rgba` и остальные помощники упаковки ARGB — на странице [Рендер 2D](../ui/render-2d.md).

## Выбор

| Метод | Тип | Описание |
|---|---|---|
| `selectable.value()` | `Entry` | выбранный сейчас entry |
| `selectable.select(entry)` | `Selectable` | выбирает его, снимает остальные (бросает ScriptException, если entry не из этой настройки) |
| `selectable.select(reference)` | `Selectable` | разрешает ссылку и выбирает (бросает ScriptException, если неизвестна) |
| `selectable.selected(entry)` | `boolean` | выбран ли этот entry; для чужих объектов false |
| `selectable.selected(reference)` | `boolean` | то же по ссылке (бросает ScriptException, если неизвестна) |
| `selectable.entry(reference)` | `Entry` | ищет по полному id, имени или последнему сегменту id (бросает ScriptException, если неизвестна) |
| `selectable.entries()` | `List<Entry>` | все варианты, порядок объявления, неизменяемая копия |
| `selectable.options()` | `List<String>` | имена вариантов, порядок объявления |
| `selectable.onChange(listener)` | `Selectable` | `Consumer<Entry>` с новым entry (только настройки скрипта) |
| `selectable.id(stableId)` | `Selectable` | ковариантный `Setting.id` |
| `selectable.visibleWhen(condition)` | `Selectable` | ковариантный `Setting.visibleWhen` |
| `combo.value()` | `List<Entry>` | выбранные entry, порядок объявления, неизменяемая копия |
| `combo.value(selected)` | `Combo` | выбирает ровно эти, снимает остальные; null снимает всё |
| `combo.has(entry)` | `boolean` | выбран ли этот entry; для чужих объектов false |
| `combo.has(reference)` | `boolean` | то же по ссылке (бросает ScriptException, если неизвестна) |
| `combo.set(entry, selected)` | `Combo` | выбирает или снимает один entry (бросает ScriptException, если entry не из этой настройки) |
| `combo.set(reference, selected)` | `Combo` | разрешает ссылку и ставит состояние (бросает ScriptException, если неизвестна) |
| `combo.entry(reference)` | `Entry` | ищет по полному id, имени или последнему сегменту id (бросает ScriptException, если неизвестна) |
| `combo.entries()` | `List<Entry>` | все варианты, порядок объявления, неизменяемая копия |
| `combo.options()` | `List<String>` | имена вариантов, порядок объявления |
| `combo.onChange(listener)` | `Combo` | `Consumer<List<Entry>>` со списком выбранных (только настройки скрипта) |
| `combo.id(stableId)` | `Combo` | ковариантный `Setting.id` |
| `combo.visibleWhen(condition)` | `Combo` | ковариантный `Setting.visibleWhen` |

Поиск по ссылке игнорирует регистр, точки, дефисы, подчёркивания и пробелы.
Сам `Entry` описан на странице [Entry со своей логикой](entries.md).

## Хоткей и кнопка

| Метод | Тип | Описание |
|---|---|---|
| `hotkey.key()` | `Key` | привязанная клавиша, `Key.UNKNOWN` если не привязана или не распознана |
| `hotkey.key(k)` | `Hotkey` | перепривязывает клавишу, модификаторы не трогает; null становится `UNKNOWN` |
| `hotkey.mods()` | `int` | битовая маска модификаторов GLFW, 0 если их нет |
| `hotkey.bound()` | `boolean` | не равна ли клавиша `UNKNOWN` |
| `hotkey.id(stableId)` | `Hotkey` | ковариантный `Setting.id` |
| `hotkey.visibleWhen(condition)` | `Hotkey` | ковариантный `Setting.visibleWhen` |
| `button.id(stableId)` | `Button` | ковариантный `Setting.id` |
| `button.visibleWhen(condition)` | `Button` | ковариантный `Setting.visibleWhen` |

У `Button` нет значения: ни `value()`, ни `onChange`, ни делегата — только действие, заданное при создании.

## Единицы

| Константа | Описание |
|---|---|
| `Postfixes.PERCENT` | рисуется как `%` |
| `Postfixes.DEGREES` | рисуется как `°` |
| `Postfixes.SECONDS` | рисуется как `s` |
| `Postfixes.TICKS` | рисуется как `t` |
| `Postfixes.BLOCKS` | рисуется как `b` |
| `Postfixes.MS` | рисуется как `ms` |
| `Postfixes.MULTIPLIER` | рисуется как `x` |
| `Postfixes.HEALTH` | рисуется как `hp` |

Постфикс принимают только `Slider` и `RangeSlider`; без него число рисуется голым.
