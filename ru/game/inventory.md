# Инвентарь и предметы

`inventory` — это `game.inventory()`: 36 основных слотов плюс четыре слота брони и вторая рука. Чтение бесплатно; любое изменение ставится в очередь перекладывания клиента и требует клиентского потока.

```kotlin
on<PrePlayerTickEvent> {
    val slot = inventory.find("golden_apple")
    if (!slot.found() || inventory.busy()) return@on
    val stack = inventory.item(slot)
    chat.print("${stack.count()} x ${stack.name()}")
    inventory.shiftClick(slot)
}
```

## Адрес слота

| Метод | Тип | Описание |
|---|---|---|
| `Slot.hotbar(index)` | `Slot` | слот хотбара по индексу (бросает ScriptException вне 0..8) |
| `Slot.inventory(index)` | `Slot` | слот основного инвентаря, меньше 9 даёт вид HOTBAR (бросает ScriptException вне 0..35) |
| `Slot.armor(slot)` | `Slot` | слот брони по `ArmorSlot` (бросает ScriptException на null) |
| `Slot.offhand()` | `Slot` | слот второй руки |
| `Slot.NONE` | `Slot` | константа «не найдено», вид NONE и индекс -1 |
| `Slot.HOTBAR_SIZE` | `int` | константа 9 |
| `Slot.MAIN_SIZE` | `int` | константа 36 |
| `slot.kind()` | `Slot.Kind` | какую часть инвентаря адресует слот |
| `slot.index()` | `int` | индекс внутри вида, -1 для NONE, порядковый номер ArmorSlot для ARMOR |
| `slot.found()` | `boolean` | вид не NONE |
| `slot.inHotbar()` | `boolean` | вид HOTBAR |
| `slot.isArmor()` | `boolean` | вид ARMOR |
| `slot.isOffhand()` | `boolean` | вид OFFHAND |
| `slot.hotbarIndex()` | `int` | индекс хотбара 0..8, -1 если это не хотбар |
| `slot.armorSlot()` | `ArmorSlot?` | слот брони, null если вид не ARMOR |
| `slot.toString()` | `String` | `none`, `hotbar N`, `inventory N`, `armor HELMET` или `offhand` |

### Slot.Kind

| Константа | Описание |
|---|---|
| `NONE` | не найденный или отсутствующий слот |
| `HOTBAR` | хотбар, индекс 0..8 |
| `INVENTORY` | основной инвентарь, индекс 9..35 |
| `ARMOR` | слот брони, индекс — порядковый номер `ArmorSlot` |
| `OFFHAND` | вторая рука, индекс 0 |

### ArmorSlot

| Константа | Описание |
|---|---|
| `HELMET` | голова, порядковый номер 0 |
| `CHESTPLATE` | грудь, порядковый номер 1 |
| `LEGGINGS` | ноги, порядковый номер 2 |
| `BOOTS` | ботинки, порядковый номер 3 |

## Чтение

| Метод | Тип | Описание |
|---|---|---|
| `inventory.held()` | `Item` | стак в основной руке, пустой предмет если рука пуста |
| `inventory.offhand()` | `Item` | стак во второй руке |
| `inventory.item(slot)` | `Item` | стак в этом слоте (бросает ScriptStateException на ненайденном слоте) |
| `inventory.armor(slot)` | `Item` | надетый стак в этом слоте брони (бросает ScriptException на null) |
| `inventory.items()` | `List<Item>` | неизменяемый список 36 основных стаков по порядку индексов |
| `inventory.selected()` | `Slot` | текущий выбранный слот хотбара |
| `inventory.count(itemId)` | `int` | суммарное количество предмета по 36 основным слотам |
| `inventory.full()` | `boolean` | среди 36 основных слотов нет пустого |
| `inventory.empty()` | `boolean` | все 36 основных слотов пусты |
| `inventory.wearing(itemId)` | `boolean` | предмет надет в одном из четырёх слотов брони |

Идентификаторы по умолчанию берут пространство `minecraft:`, так что `"mace"` и `"minecraft:mace"` — один и тот же id.
Чтение никогда не возвращает null: отсутствующий стак приходит предметом, у которого `empty()` равно true.

## Поиск

| Метод | Тип | Описание |
|---|---|---|
| `inventory.find(itemId)` | `Slot` | первый слот основного инвентаря с этим предметом, `Slot.NONE` если нет |
| `inventory.find(filter)` | `Slot` | первый слот основного инвентаря, прошедший фильтр (бросает ScriptException на null-фильтре) |
| `inventory.findByTag(tagId)` | `Slot` | первый слот основного инвентаря с этим тегом, ведущий `#` отбрасывается |
| `inventory.findInHotbar(itemId)` | `Slot` | первый слот хотбара 0..8 с этим предметом |
| `inventory.findInHotbar(filter)` | `Slot` | первый слот хотбара 0..8, прошедший фильтр |
| `inventory.findUsable(filter)` | `Slot` | первый подходящий слот, непустой и не в откате |
| `inventory.findAll(filter)` | `List<Slot>` | все подходящие слоты основного инвентаря по порядку индексов |

## Перекладывание

| Метод | Тип | Описание |
|---|---|---|
| `inventory.click(slot, rightButton)` | `void` | ставит в очередь клик PICKUP, кнопка 1 при rightButton (только главный поток) |
| `inventory.shiftClick(slot)` | `void` | ставит в очередь клик QUICK_MOVE (только главный поток) |
| `inventory.swap(slot, hotbarSlot)` | `void` | ставит в очередь SWAP в этот слот хотбара (только главный поток) (бросает ScriptException если hotbarSlot не слот хотбара) |
| `inventory.drop(slot, wholeStack)` | `void` | ставит в очередь THROW, кнопка 1 для всего стака (только главный поток) |
| `inventory.dropHeld(wholeStack)` | `void` | выбрасывает выбранный стак хотбара напрямую (только главный поток) |
| `inventory.batch(actions)` | `void` | ставит собранные действия одной последовательностью (только главный поток) |
| `inventory.busy()` | `boolean` | очередь перекладывания клиента не пуста |

### Inventory.Batch

| Метод | Тип | Описание |
|---|---|---|
| `batch.click(slot, rightButton)` | `Batch` | добавляет клик PICKUP |
| `batch.shiftClick(slot)` | `Batch` | добавляет клик QUICK_MOVE |
| `batch.swap(slot, hotbarSlot)` | `Batch` | добавляет SWAP (бросает ScriptException если hotbarSlot не слот хотбара) |
| `batch.drop(slot, wholeStack)` | `Batch` | добавляет THROW, кнопка 1 для всего стака |
| `batch.delay(ticks)` | `Batch` | добавляет паузу в игровых тиках (бросает ScriptException при ticks 0 или меньше) |
| `batch.onFinish(action)` | `Batch` | выполняет действие, когда очередь закрывается (API 2) (бросает ScriptException при null) |

Любое изменение требует открытого screen handler, а при открытом не-игровом экране слоты `ARMOR` и `OFFHAND` бросают `ScriptStateException`; батч ничего не блокирует, его очередь досылается в следующие тики.
Слоты открытого сундука или серверного меню адресуются через [Контейнеры](containers.md), смена слота в руке — через [Слоты и броню](../actions/slots.md).

## Стак

| Метод | Тип | Описание |
|---|---|---|
| `item.empty()` | `boolean` | стак пуст |
| `item.id()` | `String` | идентификатор предмета, `minecraft:air` для пустого стака |
| `item.name()` | `String` | отображаемое имя обычным текстом |
| `item.count()` | `int` | размер стака |
| `item.maxCount()` | `int` | максимальный размер стака |
| `item.damage()` | `int` | текущее значение урона прочности |
| `item.maxDamage()` | `int` | максимальная прочность, 0 если предмет не портится |
| `item.damageable()` | `boolean` | стак может получать урон прочности |
| `item.stackable()` | `boolean` | стак может держать больше одного предмета |
| `item.unbreakable()` | `boolean` | есть компонент UNBREAKABLE |
| `item.enchanted()` | `boolean` | есть хотя бы одно зачарование |
| `item.rarity()` | `Rarity` | ванильная редкость стака |
| `item.isA(itemId)` | `boolean` | идентификатор совпадает, false для пустого стака |
| `item.hasTag(tagId)` | `boolean` | предмет состоит в этом теге, ведущий `#` отбрасывается |

## Что умеет предмет

| Метод | Тип | Описание |
|---|---|---|
| `item.placeable()` | `boolean` | BlockItem, у чьего состояния по умолчанию есть коллизия |
| `item.buildable()` | `boolean` | BlockItem: полный куб, без блок-сущности, обычные трение и множители, не падающий блок и не магма (API 2) |
| `item.food()` | `boolean` | есть компонент FOOD |
| `item.nutrition()` | `int` | питательность еды, 0 если это не еда |
| `item.saturation()` | `float` | насыщение еды, 0 если это не еда |
| `item.useAction()` | `String` | действие использования: `none`, `eat`, `bow`, `block` |
| `item.disablesBlocking()` | `boolean` | компонент WEAPON сбивает блок щитом |
| `item.enchantments()` | `Map<String, Integer>` | идентификатор зачарования к уровню, порядок вставки |
| `item.enchantmentLevel(id)` | `int` | уровень по точному id, 0 если нет, без подстановки пространства |
| `item.attributeModifiers()` | `List<AttributeModifier>` | записи ATTRIBUTE_MODIFIERS, пусто если компонента нет |
| `item.containerItems()` | `List<Item>` | содержимое CONTAINER, иначе BUNDLE_CONTENTS, иначе пусто |

### AttributeModifier

| Метод | Тип | Описание |
|---|---|---|
| `modifier.attribute()` | `String` | идентификатор атрибута, к которому применяется модификатор |
| `modifier.id()` | `String` | идентификатор модификатора |
| `modifier.value()` | `double` | величина модификатора |
| `modifier.operation()` | `String` | `add_value`, `add_multiplied_base` или `add_multiplied_total` |
| `modifier.slot()` | `String` | группа слотов экипировки: `any`, `mainhand`, `head` |

## Откаты

| Метод | Тип | Описание |
|---|---|---|
| `item.onCooldown()` | `boolean` | группа отката этого стака в откате, false вне мира |
| `item.cooldownProgress()` | `float` | остаток отката при tickDelta 0, считает вниз 1..0 (API 2) |
| `item.cooldownProgress(tickDelta)` | `float` | тот же остаток, сглаженный по tickDelta, 1..0 (API 2) |
| `item.setCooldown(ticks)` | `void` | клиентский откат на столько тиков, 0 и меньше снимает его (API 2) (только главный поток) (бросает ScriptStateException вне мира) |
| `item.removeCooldown()` | `void` | снимает клиентский откат этой группы (API 2) (только главный поток) (бросает ScriptStateException вне мира) |

Откат принадлежит группе предмета, а не стаку, поэтому любой стак этого предмета отдаёт одно и то же значение.
`setCooldown` и `removeCooldown` пишут только в клиентскую копию; у сервера копия своя, и следующий пакет отката по этой группе затирает твою.

## Имена, лор, компоненты

| Метод | Тип | Описание |
|---|---|---|
| `item.displayName()` | `Text` | оформленное отображаемое имя |
| `item.customName()` | `String?` | текст компонента CUSTOM_NAME, null если его нет |
| `item.lore()` | `List<Text>` | оформленные строки LORE, пусто если их нет |
| `item.tooltip(advanced)` | `List<Text>` | строки ванильного тултипа, версия F3+H при advanced true |
| `item.tags()` | `List<String>` | идентификаторы теговых списков предмета |
| `item.hasComponent(id)` | `boolean` | компонент данных присутствует, false для неизвестного id |
| `item.component(id)` | `String?` | `toString()` значения компонента, null если его нет |
| `item.nbt()` | `String` | тег CUSTOM_DATA текстом, пустая строка если его нет |
| `item.customData(key)` | `String?` | строковое значение ключа в CUSTOM_DATA, null если его нет |

Значения `Text` — это [оформленный текст](../ui/text.md).

## Редкость

| Константа | Описание |
|---|---|
| `COMMON` | белое имя |
| `UNCOMMON` | жёлтое имя |
| `RARE` | бирюзовое имя |
| `EPIC` | светло-фиолетовое имя |

## Руки

### Hand

| Константа | Описание |
|---|---|
| `MAIN_HAND` | основная рука |
| `OFF_HAND` | вторая рука |

### Arm

| Константа | Описание |
|---|---|
| `LEFT` | левая рука |
| `RIGHT` | правая рука |

`Arm` — это `nursultan.item.Arm`, сторона основной руки из `RenderItemEvent.arm()` и пакетов клиентских настроек.
