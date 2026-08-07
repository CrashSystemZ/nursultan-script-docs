# Entry со своей логикой

Варианты у `selectable` и `combo` — это не строки, а объекты **entry**. Строки тоже работают, но entry позволяет привязать к варианту его собственное поведение и не сравнивать текст.

## Строки — когда логики нет

Если варианты просто помечают режим, пиши строками:

```kotlin
val mode by selectable("Режим", "Fast", "Smooth", selected = "Fast")

if (mode == "Fast") { ... }
```

Entry для них создадутся сами.

## Entry — когда логика есть

`entry("Имя")` создаёт вариант, к которому можно прицепить обработчики:

```kotlin
val fast = entry("Fast", true) {
    onSelect { chat.print("быстрый режим") }
    onDeselect { chat.print("больше не быстрый") }
    on<ClientTickEvent> { control.jump() }
}

val smooth = entry("Smooth") {
    on<ClientTickEvent> {
        if (player.onGround()) {
            control.jump()
        }
    }
}

val mode = selectable("Режим", fast, smooth)
```

| Обработчик | Когда вызывается |
|---|---|
| `onSelect` | вариант выбрали |
| `onDeselect` | выбрали другой (для `combo` — сняли галочку) |
| `on<Event>` | сработало это событие, но **только пока этот вариант выбран и скрипт включён** |

Второй аргумент `entry("Fast", true)` — выбран ли вариант по умолчанию. У `selectable` ровно один вариант должен быть выбран; не пометишь ни одного — выберется первый. У `combo` можно сколько угодно, в том числе ни одного.

Так режим со своей логикой пишется целиком в одном месте, а обработчик события не превращается в лестницу из `if`.

## Любое событие, как у скрипта

`on` внутри entry — это тот же `on`, что и у самого скрипта: любое событие, `priority` и `ignoreCancelled` работают, возвращается `Subscription`:

```kotlin
val silent = entry("Silent", true) {
    on<MovePacketEvent>(priority = Priority.LAST) { it.onGround(true) }
    on<AttackEvent> { chat.print("бью " + it.target().name()) }
}

val legit = entry("Legit") {
    on<JumpEvent> { chat.print("прыгнул") }
}

val mode = selectable("Режим", silent, legit)
```

Отличие одно — то самое правило выше: обработчик срабатывает, только пока его вариант выбран. Никаких подписок и отписок при переключении не происходит: слушатель регистрируется один раз при загрузке и просто молчит, пока вариант не выбран, так что хоть десяток вариантов — это ничего не стоит.

Именно это превращает `selectable` из флажка режима в набор независимых поведений: каждый вариант подписывается на то, что ему нужно, и никому не приходится разводить события руками.

## Работа с выбором

```kotlin
mode.value()              // выбранный Entry
mode.value() == fast      // сравнение по объекту, без строк
mode.selected(smooth)     // то же самое, но короче
mode.select(fast)         // выбрать
mode.entries()            // все варианты
mode.options()            // их имена строками
```

У `combo` то же самое, только про множество:

```kotlin
val items = combo("Предметы", trident, crossbow)

items.value()             // список выбранных Entry
items.has(trident)
items.set(crossbow, false)
items.value(listOf(trident))
```

## Найти entry по имени

Если объекта под рукой нет — например, работаешь с [модулем клиента](../extras/modules.md) — вариант можно достать по ссылке:

```kotlin
val targets = client.modules().get("AttackAura").combo("targets")

targets.set(targets.entry("entry.players"), true)
targets.set("players", true)          // то же самое короче
```

Ищется по полному ключу (`entry.players`), по имени и по последнему сегменту; регистр, точки, дефисы и подчёркивания не важны.

## Что нельзя

К entry встроенного модуля клиента прицепить `onSelect`, `onDeselect` или `on<Event>` нельзя — это чужая настройка, скрипт может её только читать и переключать.
