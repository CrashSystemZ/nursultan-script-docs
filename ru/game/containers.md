# Контейнеры

`container` — это экран, который сервер открыл перед тобой: сундук, шалкер, житель, меню магазина. Через него можно прочитать, что там лежит, и понажимать.

```kotlin
on<ClientTickEvent> {
    val screen = game.screen() ?: return@on
    if (screen.title().string() != "Продажа") return@on

    val slot = container.find("minecraft:cobblestone")
    if (slot.found()) {
        container.shiftClick(slot)
    }
}
```

## Нумерация слотов не такая, как в инвентаре

[`inventory`](inventory.md) адресует **твои собственные** ячейки — `Slot.hotbar(3)`, `Slot.armor(...)`. `container` адресует **открытый хэндлер**: это плоский список слотов, который разложил сервер, и в нём обычные индексы, завёрнутые в `ContainerSlot`.

Хэндлер всегда кладёт слоты самого контейнера первыми, а твой инвентарь после них:

| Диапазон | Что это |
|---|---|
| `0` … `storage() - 1` | контейнер — 54 слота у шестирядного сундука |
| `storage()` … `storage() + 26` | ряды твоего основного инвентаря |
| `storage() + 27` … `size() - 1` | твой хотбар |

```kotlin
container.syncId()      // идентификатор, который сервер выдал меню
container.size()        // все слоты, контейнер и твой инвентарь вместе
container.storage()     // сколько ведущих слотов принадлежат контейнеру
container.open()        // false, когда открыт только твой собственный инвентарь
```

Когда контейнер не открыт, это твой собственный инвентарный хэндлер, и `storage()` — это результат крафта и сетка.

`playerSlot(...)` переводит обратно, чтобы слот можно было отдать остальному API:

```kotlin
val slot = container.find("minecraft:totem_of_undying")
val mine = container.playerSlot(slot)      // Slot.NONE, если он в самом контейнере
if (mine.found()) {
    slots.using(mine) { interaction.useItem(Hand.MAIN_HAND) }
}
```

## Чтение

```kotlin
container.item(0)                          // по сырому индексу
container.item(slot)                       // по ContainerSlot
container.items()                          // всё, в порядке хэндлера

container.find("minecraft:diamond")        // первый слот, где он лежит
container.find("diamond", 0, container.storage())   // только внутри контейнера
container.find { it.enchantmentLevel("sharpness") >= 4 }
container.findAfter(10) { !it.empty() }
container.findAll { it.hasTag("swords") }

container.count("minecraft:emerald")
container.empty()                          // первый свободный слот, или NONE
container.empty(0, container.storage())
```

`item(...)` никогда не возвращает `null`: у пустого слота ты получишь пустой предмет, так что проверяй `empty()`.

## Клики

```kotlin
container.click(slot)                          // левая кнопка
container.click(slot, 1)                       // правая
container.click(slot, 0, SlotAction.QUICK_MOVE)
container.shiftClick(slot)                     // то же самое, короче
container.clickButton(0)                       // кнопка экрана: резчик, ткацкий станок, сделка жителя
```

| Действие | Что делает |
|---|---|
| `PICKUP` | взять стак / положить — по умолчанию |
| `QUICK_MOVE` | shift-клик: одним пакетом перекидывает весь стак на другую сторону |
| `SWAP` | обмен со слотом хотбара, `button` — это `0..8` |
| `THROW` | выбросить наружу |
| `PICKUP_ALL` | двойной клик, собирает одинаковые стаки |
| `QUICK_CRAFT` | жест протаскивания по слотам |
| `CLONE` | средняя кнопка, только креатив — в выживании бросает ошибку |

`QUICK_MOVE` — то, что нужно для перекладывания: один клик двигает весь стак вместо того, чтобы взять и положить дважды.

## Рецепты

`recipes` — это книга рецептов: записи, которые прислал сервер. У каждой — числовой id на эту сессию, и по нему сервер **сам раскладывает ингредиенты по сетке**, как клик по рецепту в зелёной книжке:

```kotlin
val entry = recipes.find("experience_bottle").firstOrNull()
if (entry != null && container.open()) {
    recipes.craft(entry.id(), true)    // true — крафтить сразу стак
}
```

| | |
|---|---|
| `all()` | все записи книги рецептов |
| `find(itemId)` | записи, чей результат — этот предмет |
| `craft(id[, craftAll])` | попросить сервер разложить ингредиенты; `craftAll = true` — на полный стак |

У записи есть `id()`, `resultId()`, `resultCount()`, `category()` и `craftable()` — хватает ли ингредиентов в твоём инвентаре прямо сейчас.

### Сетка и ингредиенты

Что и куда класть, тоже видно — `kind()`, `width()`, `height()`, `station()` и `ingredients()`:

```kotlin
val entry = recipes.find("crafting_table").first()
for (row in 0 until entry.height()) {
    val line = (0 until entry.width()).joinToString(" | ") { column ->
        entry.ingredients()[row * entry.width() + column].firstOrNull()?.id() ?: "-"
    }
    chat.print(line)
}
```

`ingredients()` — список слотов, и в каждом слоте список допустимых предметов: рецепт с тегом («любые доски») даёт все варианты, пустой слот сетки — пустой список.

| `kind()` | Что в `ingredients()` |
|---|---|
| `SHAPED` | сетка построчно, `width() * height()` слотов, пустые включены |
| `SHAPELESS` | ингредиенты без позиций, `width()` и `height()` — 0 |
| `FURNACE` | один слот — что переплавляется |
| `STONECUTTER` | один слот — что режется |
| `SMITHING` | три слота: шаблон, основа, добавка |
| `OTHER` | пусто |

`station()` — на каком блоке это крафтится: `"minecraft:crafting_table"`, `"minecraft:furnace"`, … Пустая строка, если сервер блок не прислал.

* `craft` кликает по рецепту в **открытом сейчас** хэндлере: сначала открой верстак — или свой инвентарь для рецептов 2×2. Результат сервер кладёт в слот результата, забирай его сам — `shiftClick`.
* `id` живёт одну сессию: сервер раздаёт индексы при входе. Не сохраняй его между заходами — ищи запись заново по `resultId()`.
* Сервер разложит только рецепт, который сам знает и прислал в книгу. Кастомные крафты плагинов, которых в книге рецептов нет, так не крафтятся.

## Последовательности и задержки

Одиночный клик уходит сразу. Когда нужно несколько по порядку — и серверу нужен тик между ними, чтобы успеть, — ставь их одной очередью:

```kotlin
container.batch {
    it.click(from, 0, SlotAction.SWAP)
    it.delay(1)
    it.click(to, 0, SlotAction.SWAP)
    it.delay(1)
    it.click(from, 0, SlotAction.SWAP)
}
```

Весь батч — одна очередь: клиент отправляет первый клик, ждёт задержку в **игровых тиках**, отправляет следующий. Ничего не блокируется — твой обработчик возвращается сразу, а очередь досылается сама в следующие тики.

| | |
|---|---|
| `click(slot[, button[, action]])` | как выше |
| `shiftClick(slot)` | |
| `swap(slot, hotbarIndex)` | `hotbarIndex` — это `0..8` |
| `drop(slot, wholeStack)` | |
| `delay(ticks)` | подождать перед следующим шагом, от 1 тика |
| `onFinish { ... }` | сработает, когда батч закончился; пустой батч отбрасывается вместе с ним |

`container.busy()` — `true`, пока очередь ещё досылается: твоя или собственная клиентская. Дождись её, прежде чем начинать новую:

```kotlin
on<ClientTickEvent> {
    if (container.busy()) return@on
    val slot = container.find { !it.empty() }
    if (slot.found()) {
        container.shiftClick(slot)
    }
}
```

Клиент не замедляет твои клики за тебя. Что безопасно, зависит от сервера, так что дозируй сам через `delay` и `busy()` — мгновенный ChestStealer это самый быстрый способ подставить своих пользователей под бан.

## Что учесть

* **Экран может закрыться между чтением и кликом.** Слота, который ты нашёл тик назад, может уже не быть; клик проверяется по тому хэндлеру, который открыт прямо сейчас, и бросает ошибку, если индекс вне диапазона.
* Клики работают **только на клиентском потоке**. Из обработчика пакетов оборачивай в `onClientThread { }`.
* Брони и второй руки в чужом хэндлере нет. `inventory.click(Slot.armor(...))` при открытом сундуке скажет об этом, а не кликнет наугад.
* Закрыть экран — `game.closeScreen()`.
* Серверы переставляют свои меню между тиками. Ищи по предмету, а не по индексу, который видел в прошлый раз.
