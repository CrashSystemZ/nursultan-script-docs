# Контейнеры

`container` — это `game.container()`: открытый хэндлер экрана как один плоский список слотов. Индексы `0..storage()-1` — слоты самого контейнера, дальше 27 слотов инвентаря, дальше 9 слотов хотбара; любой метод бросает `ScriptStateException`, когда нет мира или нет хэндлера экрана.

```kotlin
on<ClientTickEvent> {
    if (!container.open() || container.busy()) return@on
    val slot = container.find("minecraft:diamond", 0, container.storage())
    if (!slot.found()) return@on
    container.batch {
        it.shiftClick(slot)      // по одному шагу за тик
        it.delay(2)
    }
}
```

## Нумерация слотов

| Метод | Тип | Описание |
|---|---|---|
| `container.syncId()` | `int` | sync id открытого хэндлера экрана |
| `container.size()` | `int` | всего слотов в открытом хэндлере |
| `container.storage()` | `int` | `size() - 36`, не меньше 0 — слоты самого контейнера |
| `container.open()` | `boolean` | true, когда sync id не ноль, то есть открыт не свой инвентарь |
| `container.playerSlot(slot)` | `Slot` | индекс хэндлера как слот инвентаря; `Slot.NONE` вне игроцкой части |

| Метод | Тип | Описание |
|---|---|---|
| `ContainerSlot.NONE` | `ContainerSlot` | константа с индексом -1, значит «не найдено» |
| `ContainerSlot.of(index)` | `ContainerSlot` | оборачивает сырой индекс хэндлера (бросает `ScriptException` при index < 0) |
| `slot.index()` | `int` | сырой индекс слота хэндлера, -1 у `NONE` |
| `slot.found()` | `boolean` | индекс 0 или больше |
| `slot.equals(other)` | `boolean` | true, когда у другого `ContainerSlot` тот же индекс |
| `slot.hashCode()` | `int` | сам индекс |
| `slot.toString()` | `String` | `none` или `container <index>` |

`playerSlot` переводит первые 27 слотов игроцкой части в `Slot.inventory(9..35)`, последние 9 — в `Slot.hotbar(0..8)`.
`Slot` и `Item` описаны на странице [Инвентарь и предметы](inventory.md).

## Чтение

| Метод | Тип | Описание |
|---|---|---|
| `container.item(slot)` | `Item` | стак в этом `ContainerSlot`; пустой предмет при null или вне диапазона |
| `container.item(index)` | `Item` | стак по сырому индексу; пустой предмет вне диапазона |
| `container.items()` | `List<Item>` | все слоты в порядке хэндлера |
| `container.find(itemId)` | `ContainerSlot` | первый слот с этим id предмета; `NONE`, если нет |
| `container.find(itemId, from, to)` | `ContainerSlot` | то же, но в границах диапазона индексов |
| `container.find(filter)` | `ContainerSlot` | первый слот, прошедший фильтр; `NONE` при null-фильтре |
| `container.find(from, to, filter)` | `ContainerSlot` | первый подходящий слот внутри диапазона |
| `container.findAfter(from, filter)` | `ContainerSlot` | первый подходящий слот от этого индекса до конца |
| `container.findAll(filter)` | `List<ContainerSlot>` | все подходящие слоты; пусто при null-фильтре |
| `container.count(itemId)` | `int` | сумма размеров стаков этого предмета по хэндлеру |
| `container.count(itemId, from, to)` | `int` | то же, но по диапазону индексов |
| `container.empty()` | `ContainerSlot` | первый пустой слот или `NONE` |
| `container.empty(from, to)` | `ContainerSlot` | первый пустой слот внутри диапазона |

Диапазоны индексов полуоткрытые и зажимаются в `0..size()`.
Id предметов по умолчанию получают неймспейс `minecraft:`; пустой id бросает `ScriptException`.

## Клики

| Метод | Тип | Описание |
|---|---|---|
| `container.click(slot)` | `void` | левый клик, кнопка 0, `PICKUP` (только главный поток) (бросает `ScriptException`, если слот `NONE` или вне хэндлера) |
| `container.click(slot, button)` | `void` | клик указанной кнопкой мыши, `PICKUP` (только главный поток) |
| `container.click(slot, button, action)` | `void` | клик с явным действием слота (только главный поток) |
| `container.shiftClick(slot)` | `void` | клик кнопкой 0 и `QUICK_MOVE` (только главный поток) |
| `container.clickButton(button)` | `void` | кнопка хэндлера: резчик, ткацкий станок, сделка жителя (только главный поток) |
| `container.busy()` | `boolean` | true, пока в клиентской очереди перекладывания есть действия |

| Константа | Описание |
|---|---|
| `PICKUP` | обычный левый/правый клик; он же подстановка при null-действии |
| `QUICK_MOVE` | перенос shift-кликом |
| `SWAP` | обмен с хотбаром, button — индекс хотбара 0..8 |
| `CLONE` | клон средней кнопкой (бросает `ScriptException` вне креатива) |
| `THROW` | выброс; кнопка 1 выбрасывает весь стак, 0 — один предмет |
| `QUICK_CRAFT` | протаскивание по слотам |
| `PICKUP_ALL` | двойной клик, собирает всё одинаковое |

`busy()` учитывает и собственные перекладывания клиента, не только скриптовые.

## Последовательности

| Метод | Тип | Описание |
|---|---|---|
| `container.batch(actions)` | `void` | ставит шаги в очередь перекладывания, она досылается в следующие тики (только главный поток) (бросает `ScriptException` при null) |

| Метод | Тип | Описание |
|---|---|---|
| `it.click(slot)` | `Batch` | ставит в очередь кнопку 0 / `PICKUP` |
| `it.click(slot, button)` | `Batch` | ставит клик этой кнопкой мыши / `PICKUP` |
| `it.click(slot, button, action)` | `Batch` | ставит клик с явным действием слота |
| `it.shiftClick(slot)` | `Batch` | ставит кнопку 0 / `QUICK_MOVE` |
| `it.swap(slot, hotbarIndex)` | `Batch` | ставит `SWAP` на слот хотбара (бросает `ScriptException`, если hotbarIndex вне 0..8) |
| `it.drop(slot, wholeStack)` | `Batch` | ставит `THROW`; кнопка 1 — весь стак, 0 — один предмет |
| `it.delay(ticks)` | `Batch` | ставит ожидание в игровых тиках (бросает `ScriptException` при ticks <= 0) |
| `it.onFinish(action)` | `Batch` | вызов на хуке закрытия очереди; ванильный процессор зовёт сразу (API 2) (бросает `ScriptException` при null) |

Батч, в который не поставили ни клика, ни задержки, отбрасывается, и его `onFinish` не сработает.

## Рецепты

| Метод | Тип | Описание |
|---|---|---|
| `recipes.all()` | `List<RecipeEntry>` | все записи книги рецептов в порядке книги (API 2) (только главный поток) |
| `recipes.find(resultItemId)` | `List<RecipeEntry>` | записи, чей результат — этот id предмета (API 2) (только главный поток) |
| `recipes.craft(recipeId)` | `void` | кликает рецепт в открытом экране крафта, craftAll false (API 2) (только главный поток) |
| `recipes.craft(recipeId, craftAll)` | `void` | сервер раскладывает ингредиенты в открытую сетку; craftAll — на полный стак (API 2) (только главный поток) |

## Запись рецепта

| Метод | Тип | Описание |
|---|---|---|
| `entry.id()` | `int` | сетевой id рецепта, действителен только для текущего подключения (API 2) |
| `entry.resultId()` | `String` | registry id показанного предмета-результата (API 2) |
| `entry.resultCount()` | `int` | размер стака показанного результата (API 2) |
| `entry.category()` | `String` | registry id категории книги рецептов; `""`, если неизвестна (API 2) |
| `entry.craftable()` | `boolean` | ингредиенты сейчас есть в инвентаре (API 2) |
| `entry.kind()` | `RecipeKind` | по какой форме разложен `ingredients()` (API 2) |
| `entry.width()` | `int` | ширина сетки; 0 у видов кроме `SHAPED` (API 2) |
| `entry.height()` | `int` | высота сетки; 0 у видов кроме `SHAPED` (API 2) |
| `entry.station()` | `String` | registry id предмета-станции крафта; `""`, если не прислан (API 2) |
| `entry.ingredients()` | `List<List<Item>>` | по слоту — список допустимых предметов; пустой список у пустой клетки (API 2) |

| Константа | Описание |
|---|---|
| `SHAPED` | сетка построчно, `width() * height()` клеток, пустые включены (API 2) |
| `SHAPELESS` | ингредиенты без позиций, width и height равны 0 (API 2) |
| `FURNACE` | один слот ингредиента, вход переплавки (API 2) |
| `STONECUTTER` | один слот ингредиента, вход резки (API 2) |
| `SMITHING` | три слота ингредиентов: шаблон, основа, добавка (API 2) |
| `OTHER` | нераспознанное отображение; `ingredients()` пуст (API 2) |
