# Модули клиента

`client.modules()` даёт доступ ко встроенным модулям клиента и их плоским деревьям настроек. Любая ссылка — на модуль, настройку или entry — сравнивается без `-`, `_`, `.` и пробелов и без учёта регистра, поэтому `AttackAura`, `attack aura` и `attack-aura` попадают в один и тот же модуль.

```kotlin
val aura = client.modules().get("AttackAura")

chat.print(aura.displayName() + " fov " + aura.slider("fov").value())
aura.slider("fov").value(120f)
if (!aura.enabled()) {
    aura.toggle()
}
```

## Найти модуль

**`Modules`**

| Метод | Тип | Описание |
|---|---|---|
| `client.modules().all()` | `List<Module>` | все модули в порядке реестра — том же, что в меню |
| `client.modules().exists(name)` | `boolean` | true, когда по этой ссылке есть модуль |
| `client.modules().find(name)` | `Module?` | найденный модуль, null если совпадений нет или имя пустое |
| `client.modules().get(name)` | `Module` | тот же поиск (бросает ScriptException, если совпадений нет) |

Поиск идёт сначала по имени в реестре, потом по имени с пробелами из меню; список неизменяемый, а модули `DEVELOPMENT` попадают в него только у администраторов.
Обёртка кешируется на модуле, поэтому повторный поиск того же модуля возвращает тот же экземпляр `Module`.

## Модуль

**`Module`**

| Метод | Тип | Описание |
|---|---|---|
| `module.name()` | `String` | имя в реестре, без пробелов, например `AttackAura` |
| `module.displayName()` | `String` | имя с пробелами перед заглавными, например `Attack Aura` |
| `module.bind()` | `Bind` | бинд-переключатель модуля — [Клавиши и бинды](../actions/keys.md) |
| `module.enabled()` | `boolean` | текущее состояние переключателя |
| `module.setEnabled(value)` | `void` | ставит состояние; игнорируется, если оно уже такое |
| `module.toggle()` | `void` | переключает состояние |

Обе записи выполняются сразу на клиентском потоке, а с любого другого ставятся в его очередь; модуль может отказаться от переключения в своём коде включения/выключения.
Любая смена состояния рассылает `ModuleToggleEvent` — [Список событий](../events/reference.md).

## Его настройки

**`Module`**

| Метод | Тип | Описание |
|---|---|---|
| `module.settings()` | `List<Setting>` | плоское дерево в глубину, вложенные включены, снимок при первом обращении |
| `module.setting(reference)` | `Setting?` | по стабильному id, потом полному имени, потом последнему сегменту; null если нет |
| `module.checkBox(reference)` | `CheckBox` | типизированный поиск (бросает ScriptException, если настройки нет или у неё другой тип) |
| `module.slider(reference)` | `Slider` | типизированный поиск (бросает ScriptException, если настройки нет или у неё другой тип) |
| `module.rangeSlider(reference)` | `RangeSlider` | типизированный поиск (бросает ScriptException, если настройки нет или у неё другой тип) |
| `module.input(reference)` | `Input` | типизированный поиск (бросает ScriptException, если настройки нет или у неё другой тип) |
| `module.selectable(reference)` | `Selectable` | типизированный поиск (бросает ScriptException, если настройки нет или у неё другой тип) |
| `module.combo(reference)` | `Combo` | типизированный поиск (бросает ScriptException, если настройки нет или у неё другой тип) |
| `module.colorPicker(reference)` | `ColorPicker` | типизированный поиск (бросает ScriptException, если настройки нет или у неё другой тип) |
| `module.hotkey(reference)` | `Hotkey` | типизированный поиск (бросает ScriptException, если настройки нет или у неё другой тип) |

Члены этих типов настроек — на странице [Виды настроек](../settings/types.md); у заголовка группы нет скриптового типа, он приходит обычным `Setting`.
У встроенной настройки `name()` — это часть locale-ключа после `.setting.`, а `id()` — весь ключ (`module.attackaura.setting.aim-range`); правый клик по настройке в меню копирует этот ключ, пока включён режим разработки.

## Чего настройки модуля не умеют

| Метод | Тип | Описание |
|---|---|---|
| `setting.id(stableId)` | `Setting` | бросает ScriptException; id — это locale-ключ модуля |
| `setting.visibleWhen(condition)` | `Setting` | бросает ScriptException; видимость в меню принадлежит клиенту |
| `onChange(listener)` | `CheckBox` `Slider` `RangeSlider` `Input` `Selectable` `Combo` `ColorPicker` | бросает ScriptException у всех семи типов настроек |
| `postfix(unit)` | `Slider` `RangeSlider` | бросает ScriptException; единицу задаёт модуль |
| `entry.onSelect(action)` | `Entry` | бросает ScriptException у entry модуля |
| `entry.onDeselect(action)` | `Entry` | бросает ScriptException у entry модуля |
| `entry.on(type, options, handler)` | `Subscription` | бросает ScriptException у entry модуля |

Чтение и запись значений работают у любой встроенной настройки: `slider.value(v)` и `rangeSlider.value(from, to)` зажимаются в собственные `min()..max()` модуля, а каждая запись уходит на клиентский поток.
Entry модуля по-прежнему переключается через свой `Selectable`/`Combo` — [Entry со своей логикой](../settings/entries.md).
