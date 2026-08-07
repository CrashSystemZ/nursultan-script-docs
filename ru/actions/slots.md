# Слоты и броня

## Сменить предмет в руке

Прямая смена слота видна и тебе, и серверу:

```kotlin
slots.selectSilently(Slot.hotbar(3))
slots.sync()                        // сообщить серверу текущий слот
```

Но чаще нужно другое: взять предмет на один удар и сразу вернуть всё как было. Для этого есть `using`:

```kotlin
val mace = inventory.findInHotbar("mace")
if (mace.found()) {
    slots.using(mace) {
        interaction.attack(target)
    }
}
```

Внутри блока предмет уже в руке, после блока слот вернётся сам — даже если внутри что-то упало с ошибкой.

## Когда нужен контроль вручную

`slots.select(...)` возвращает объект, которым можно управлять возвратом:

```kotlin
val held = slots.select(Slot.hotbar(5))

held.originalSlot()      // где были
held.slot()              // где сейчас

held.restore()           // вернуть немедленно
held.restoreWhenSafe()   // вернуть, когда это не помешает (например, после замаха)
held.keep()              // не возвращать, остаться на новом слоте
```

`restoreWhenSafe()` — то, что нужно в бою: мгновенный возврат может съесть удар.

## Броня

```kotlin
val best = armor.bestSlotFor(ArmorSlot.CHESTPLATE)
if (best.found()) {
    // в этом слоте лежит нагрудник лучше надетого
}

armor.isBetterThanEquipped(ArmorSlot.HELMET, someItem)
```

`bestSlotFor` учитывает защиту, чары и прочность и возвращает `Slot.NONE`, если ничего лучше надетого нет.

## Надеть найденное

Как именно надевать — зависит от того, где лежит предмет:

```kotlin
fun equip(slot: Slot, armorSlot: ArmorSlot) {
    // в хотбаре — просто использовать
    if (slot.inHotbar()) {
        slots.using(slot) { interaction.useItem(Hand.MAIN_HAND) }
        return
    }
    // слот брони пуст — хватит шифт-клика
    if (inventory.armor(armorSlot).empty()) {
        inventory.shiftClick(slot)
        return
    }
    // иначе меняем через свободную ячейку хотбара
    val hotbar = Slot.hotbar(inventory.selected().index() % 8 + 1)
    inventory.batch {
        it.swap(slot, hotbar)
        it.delay(1)
        it.swap(Slot.armor(armorSlot), hotbar)
        it.delay(1)
        it.swap(slot, hotbar)
    }
}
```

## Не спеши

Перекладывать инвентарь каждый тик — верный способ поругаться с сервером. Ставь между действиями задержку и не лезь, пока клиент занят:

```kotlin
val sinceSwap = timer()

on<PrePlayerTickEvent> {
    if (inventory.busy()) return@on
    if (!sinceSwap.passed(3)) return@on

    val best = armor.bestSlotFor(ArmorSlot.HELMET)
    if (!best.found()) return@on

    equip(best, ArmorSlot.HELMET)
    sinceSwap.reset()
}
```

Про `timer()` — в [Таймерах и задачах](../extras/tasks.md).
