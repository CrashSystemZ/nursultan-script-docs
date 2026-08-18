# Entry со своей логикой

`entry(name) { }` — это один вариант `selectable` или `combo`, который несёт собственные колбэки и обработчики событий. Они работают, только пока этот вариант выбран и скрипт включён.

```kotlin
val fast = entry("Fast", true) {
    onSelect { chat.print("fast mode") }
    on<ClientTickEvent> { control.jump() }
}
val smooth = entry("Smooth")

val mode = selectable("Mode", fast, smooth)
```

## Строки или entry

| Метод | Тип | Описание |
|---|---|---|
| `selectable("Mode", "A", "B")` | `Selectable` | варианты строками, entry создаются за тебя |
| `selectable("Mode", entry("A") { }, entry("B") { })` | `Selectable` | варианты объектами entry со своей логикой |
| `entry(name, selected, body)` | `Entry` | создаёт вариант, `selected` — выбран ли сразу (бросает ScriptException при пустом имени) |

У `combo` те же две формы. Если у `selectable` не отмечен ни один вариант, выбирается первый; у `combo` может не быть ни одного.
Все перегрузки `selectable` и `combo` перечислены на странице [Виды настроек](types.md).

## Внутри блока

| Метод | Тип | Описание |
|---|---|---|
| `name` | `String` | имя варианта, переданное в `entry()` |
| `selected` | `Boolean` | выбран ли этот вариант прямо сейчас |
| `on<E>(ignoreCancelled) { }` | `Subscription` | подписка; обработчик работает, только пока вариант выбран |
| `on(type, ignoreCancelled) { }` | `Subscription` | то же, класс события задан явно |
| `on<E>(priority, ignoreCancelled) { }` | `Subscription` | (устарело, убери аргумент) |
| `on(type, priority, ignoreCancelled) { }` | `Subscription` | (устарело, убери аргумент) |
| `onSelect { }` | `Unit` | вызывается, когда вариант становится выбранным |
| `onDeselect { }` | `Unit` | вызывается, когда вариант перестаёт быть выбранным |

Подписка регистрируется один раз при загрузке и живёт при переключении вариантов; снятие выбора фильтрует обработчик при рассылке, а не отписывает его.
`ignoreCancelled`, отмена события и отписка работают так же, как на странице [Подписка на события](../events/basics.md).

## Объект Entry

| Метод | Тип | Описание |
|---|---|---|
| `name()` | `String` | имя варианта; у entry модулей срезается префикс `entry.` |
| `id()` | `String` | ключ локализации `entry.<scriptId>.<name>`, нижний регистр, пробелы в дефисы |
| `selected()` | `boolean` | выбран ли этот вариант сейчас |
| `onSelect(action)` | `Entry` | добавляет действие на переход выбора в true |
| `onDeselect(action)` | `Entry` | добавляет действие на переход выбора в false |
| `on(type, handler)` | `Subscription` | подписка с `EventOptions.DEFAULT` |
| `on(type, options, handler)` | `Subscription` | подписка; работает, только пока вариант выбран и скрипт включён |

`onSelect`/`onDeselect` срабатывают только после того, как entry попал в `selectable` или `combo`, и только на реальном изменении — состояние при создании не сообщается.
Чтение и переключение выбора (`value()`, `select()`, `has()`, `entry(reference)`) — на странице [Виды настроек](types.md).

## Entry модулей клиента

`onSelect(...)`, `onDeselect(...)` и `on(...)` бросают `ScriptException` на entry, который принадлежит встроенному модулю клиента.
`name()`, `id()` и `selected()` на нём работают, а переключается он через свой `Selectable`/`Combo` — см. [модули клиента](../extras/modules.md).
