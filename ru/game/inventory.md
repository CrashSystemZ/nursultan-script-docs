# Инвентарь и предметы

`inventory` — это **твой собственный** инвентарь: хотбар, основные ряды, броня, вторая рука. Сундук или серверное меню — другая история с другой нумерацией слотов, она живёт в [Контейнерах](containers.md).

## Слоты

Слот — это адрес ячейки, а не сам предмет. Собрать его можно так:

```kotlin
Slot.hotbar(0)                  // первая ячейка хотбара, 0..8
Slot.inventory(20)              // ячейка основного инвентаря, 0..35
Slot.armor(ArmorSlot.HELMET)
Slot.offhand()
Slot.NONE                       // «не нашлось»
```

Всё, что ищет предмет, возвращает слот — и он может оказаться пустым:

```kotlin
val slot = inventory.find("minecraft:golden_apple")
if (!slot.found()) return

slot.inHotbar()
slot.hotbarIndex()
slot.isArmor()
slot.armorSlot()
```

## Что лежит

```kotlin
inventory.held()                        // в руке
inventory.offhand()
inventory.item(Slot.hotbar(3))
inventory.armor(ArmorSlot.CHESTPLATE)
inventory.items()                       // весь инвентарь списком
inventory.selected()                    // выбранный слот хотбара

inventory.count("minecraft:arrow")
inventory.wearing("minecraft:elytra")
inventory.full()
inventory.empty()
```

## Поиск

```kotlin
inventory.find("minecraft:mace")                 // по всему инвентарю
inventory.findInHotbar("minecraft:mace")         // только хотбар
inventory.findByTag("swords")                    // по тегу предметов
inventory.find { it.enchantmentLevel("sharpness") >= 4 }
inventory.findInHotbar { it.disablesBlocking() }
inventory.findUsable { it.food() }               // то, что можно применить прямо сейчас
inventory.findAll { it.damageable() }            // все подходящие
```

Идентификатор можно писать и без `minecraft:` — `"mace"` работает так же.

## Предмет

```kotlin
item.empty()                    // пустая ячейка
item.id()                       // "minecraft:diamond_sword"
item.name()                     // как называется в игре
item.count()  item.maxCount()
item.isA("diamond_sword")
item.hasTag("swords")

item.damage()  item.maxDamage()
item.damageable()
item.unbreakable()
item.stackable()

item.enchanted()
item.enchantments()             // все чары
item.enchantmentLevel("sharpness")

item.food()
item.nutrition()
item.saturation()

item.placeable()                // блок, на котором можно стоять: не факел, не табличка

item.rarity()
item.onCooldown()               // предмет в откате
item.disablesBlocking()         // топором можно сбить щит

item.useAction()                // "eat", "drink", "bow", "block", "none"...
item.tags()                     // все теги предметов, в которых он состоит
```

## Что сервер на нём написал

Серверы кладут почти всё в имя и лор: цену, продавца, откат, какой это кит. И то и другое приходит [оформленным текстом](../ui/text.md):

```kotlin
item.displayName()              // Text: имя так, как его рисует тултип
item.customName()               // String: имя из наковальни, или null
item.lore()                     // List<Text>: строки лора
item.tooltip(false)             // List<Text>: весь тултип, как нарисован
```

`tooltip(true)` — версия по F3+H, с идентификатором предмета и числами прочности.

Разбор лора — обычный способ прочитать магазин:

```kotlin
val price = inventory.held().lore()
    .map { it.string() }
    .firstOrNull { it.startsWith("Цена:") }
```

## Атрибуты и компоненты

```kotlin
for (modifier in item.attributeModifiers()) {
    modifier.attribute()        // "minecraft:attack_damage"
    modifier.value()
    modifier.operation()        // "add_value", "add_multiplied_base", "add_multiplied_total"
    modifier.slot()             // "mainhand", "head", "any"...
}
```

Шалкеры и мешочки носят содержимое с собой, так что коробку можно прочитать не открывая:

```kotlin
val inside = item.containerItems()
chat.print("в коробке стаков: " + inside.size)
```

Всё, что предмет хранит, лежит в **компоненте**. Нужные разобраны выше; для остального есть общий вход:

```kotlin
item.hasComponent("minecraft:trim")
item.component("minecraft:dyed_color")   // читаемый дамп, форма зависит от компонента
```

`component(...)` отдаёт строку, потому что у каждого компонента своя форма. Годится, чтобы проверить и залогировать, но не чтобы разбирать.

Серверы, которые держат на предмете скрытые данные, пользуются custom data:

```kotlin
item.nbt()                      // весь тег custom data текстом
item.customData("shopId")       // один строковый ключ из него, или null
```

Проверять пустоту нужно всегда: `inventory.item(...)` не возвращает `null`, он возвращает пустой предмет.

```kotlin
val held = inventory.held()
if (held.empty()) return
```

## Перекладывание

```kotlin
inventory.click(slot, false)               // false — левая кнопка
inventory.shiftClick(slot)
inventory.swap(slot, Slot.hotbar(8))
inventory.drop(slot, true)                 // true — весь стак
inventory.dropHeld(false)
```

Если действий несколько, объединяй их в один пакет — так меньше шансов, что сервер увидит рассинхрон:

```kotlin
val hotbar = Slot.hotbar(8)

inventory.batch {
    it.swap(slot, hotbar)
    it.delay(1)
    it.swap(Slot.armor(ArmorSlot.CHESTPLATE), hotbar)
    it.delay(1)
    it.swap(slot, hotbar)
}
```

`delay(ticks)` ставит паузу между двумя шагами батча, считается в **игровых тиках**. Батч ничего не блокирует: обработчик возвращается сразу, а очередь досылается в следующие тики. Обмену брони из трёх шагов эти паузы нужны — без них сервер видит три клика в одном тике, и средний попадает в слот, содержимое которого он ещё не обновил.

## Не наступай сам себе на ногу

`inventory.busy()` показывает, что клиент прямо сейчас двигает предметы — свои или чужие. Пока он `true`, лучше не лезть:

```kotlin
on<PrePlayerTickEvent> {
    if (inventory.busy()) return@on
    // теперь можно перекладывать
}
```

Смена слота в руке и подбор лучшей брони — это отдельная тема, смотри [Слоты и броня](../actions/slots.md).
