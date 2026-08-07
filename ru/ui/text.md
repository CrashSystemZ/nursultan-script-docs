# Оформленный текст

`Text` — это сообщение в чат с цветом, кликами и ховерами. Собираешь его через `text` и отдаёшь туда, куда идут сообщения.

```kotlin
chat.print(text.literal("готово").color(Colors.GREEN))
```

Этот же тип приходит обратно из игры: заголовки экранов, лор предметов, строки табло и ряды таблиста — это `Text`, а не голые строки.

## Как собрать

| Конструктор | Что делает |
|---|---|
| `text.literal("hi")` | фиксированный текст |
| `text.empty()` | пустой корень, на который вешают детей |
| `text.translatable("block.minecraft.stone")` | текст, который переведёт клиент |
| `text.keybind("key.jump")` | клавиша, которую забиндил игрок |
| `text.legacy("§aok")` | разбирает `§`-коды в настоящий стиль |
| `text.fromJson(json)` | разбирает формат провода |

`translatable` подставляет аргументы в `%s`, и аргументом может быть другой `Text`:

```kotlin
text.translatable("chat.type.text", text.literal("Notch").color(Colors.CYAN), "привет")
```

## Оформление

Каждый вызов оформления возвращает тот же объект, поэтому они цепочкой:

```kotlin
val label = text.literal("МАЛО ХП")
    .color(Colors.RED)
    .bold(true)
    .italic(false)
```

| Метод | |
|---|---|
| `color(argb)` | цвет; используется только RGB, у чата нет альфы |
| `bold(v)`, `italic(v)`, `underlined(v)`, `strikethrough(v)`, `obfuscated(v)` | ванильные переключатели |
| `font(id)` | шрифтовой провайдер, `"minecraft:default"` и такие же |
| `append(other)`, `append("текст")` | добавить ребёнка |
| `insertion(v)` | что вставится в поле ввода по shift-клику |

Стиль, заданный родителю, наследуется детьми, добавленными после него, — ровно как в ванили:

```kotlin
val line = text.empty()
    .append(text.literal("[").color(Colors.GRAY))
    .append(text.literal("убийство").color(Colors.RED).bold(true))
    .append(text.literal("] ").color(Colors.GRAY))
    .append(text.literal(target.name()))

chat.print(line)
```

## Клики и ховеры

```kotlin
chat.print(
    text.literal("[телепорт]")
        .color(Colors.CYAN)
        .runCommand("/spawn")
        .hoverText("на спавн")
)
```

| Метод | Что делает клик |
|---|---|
| `runCommand(cmd)` | выполняет, как будто игрок набрал сам |
| `suggestCommand(cmd)` | кладёт в поле ввода, не отправляя |
| `openUrl(url)` | открывает ссылку — только `http` и `https` |
| `copyToClipboard(v)` | копирует в буфер обмена |

| Метод | Что показывает ховер |
|---|---|
| `hoverText(text)` / `hoverText("...")` | подсказку |
| `hoverItem(item)` | карточку предмета, как при наведении на предмет в чате |
| `hoverEntity(entity)` | имя, тип и uuid |

Клик срабатывает только у сообщения, которое правда лежит в чате, — `chat.print` и подобные. Текст, нарисованный на HUD, — это картинка, там нажимать нечего.

## Чтение

```kotlin
val title = game.screen()?.title ?: return
title.string()      // "Сундук", оформление выброшено
title.toJson()      // формат провода
title.copy()        // независимая копия, которую можно переоформить
```

`string()` — то, что нужно для сравнений: это тот же текст, который видит игрок.

```kotlin
if (game.screen()?.title()?.string() == "Продажа") { ... }
```

## Что учесть

* **`Text` изменяемый.** `color(...)` меняет сам объект и возвращает его, а не создаёт новый. Если ты держишь `Text` и переоформляешь его, изменение увидят все, кто держит эту же ссылку. Когда это важно — `copy()`.
* `chat.print` принимает что угодно: `String`, число, `Text`. Оформление сохраняет только `Text`.
* Цвет — `0xAARRGGBB`, как и везде в API, но байт альфы чат игнорирует.
* `text.legacy(...)` понимает только `§`, но не `&`. С сервера строки приходят с `§`; когда цвета пишешь сам — используй построитель.
