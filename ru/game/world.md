# Мир и блоки

`world` — это текущий мир. Работает, только пока ты в игре, так что оборачивай в `whenInGame` или проверяй `inGame`.

## Что за мир

```kotlin
world.dimension()        // "minecraft:overworld" и т.п.
world.timeTicks()        // сколько мир существует
world.dayTimeTicks()     // время суток, 0..24000
world.raining()
world.thundering()
world.rainGradient()     // 0..1, насколько сильно идёт прямо сейчас
world.thunderGradient()  // 0..1, то же самое про грозу
```

`raining()` переключается в момент начала погоды, а градиент нарастает несколько секунд — именно им удобно плавно гасить эффект.

## Блоки

Блок берётся по координатам или по точке:

```kotlin
val under = world.block(player.x().toInt(), player.y().toInt() - 1, player.z().toInt())
val here = world.block(player.position())
```

Что можно про него спросить:

```kotlin
under.id()               // "minecraft:stone"
under.name()             // как называется в игре
under.x()  under.y()  under.z()
under.position()
under.box()              // его коробка в мире

under.air()
under.solid()
under.liquid()
under.opaque()
under.blocksMovement()
under.replaceable()      // на его месте можно поставить свой блок: воздух, вода, трава

under.luminance()        // сколько света излучает
under.hardness()         // как долго ломать
under.blastResistance()
under.slipperiness()     // лёд скользкий

under.fullCube()
under.hasCollision()
under.hasBlockEntity()   // сундук, табличка, шалкер
under.emitsRedstone()
under.toolRequired()     // голыми руками ничего не выпадет
under.burnable()
under.randomTicks()
under.pistonBehavior()   // "normal", "destroy", "block", "push_only"
under.velocityMultiplier()      // душевой песок замедляет
under.jumpVelocityMultiplier()  // мёд укорачивает прыжок
under.opacity()          // сколько света съедает
under.mapColor()         // его цвет на карте, 0xAARRGGBB
```

## Состояние блока и теги

Идентификатор блока говорит, *что* это. Состояние говорит, *как* оно поставлено. Именно там живут «открыт ли сундук», «куда повёрнута лестница» и «сколько налито в котёл»:

```kotlin
val block = world.block(position)

block.properties()          // все свойства как имя -> значение, и то и другое строки
block.property("facing")    // одно из них, или null, если у блока такого свойства нет
```

```kotlin
if (block.property("open") == "true") {
    chat.print("люк открыт")
}
```

Значения всегда строки, потому что набор свойств зависит от блока: `"true"`, `"north"`, `"5"`. Сравнивай их как текст, не жди чисел.

Теги группируют блоки так же, как их группирует игра:

```kotlin
block.tags()                     // все теги, в которых блок состоит
block.hasTag("mineable/pickaxe")
block.hasTag("minecraft:logs")   // префикс необязателен
```

## Блок-сущности

Сундук, табличка, волт, шалкер — такие блоки несут данные, о которых ни идентификатор, ни состояние ничего не говорят. `blockEntity()` доводит до них, а на обычном блоке возвращает `null`:

```kotlin
val vault = world.block(position).blockEntity() ?: return

vault.type()          // "minecraft:vault"
vault.x()  vault.y()  vault.z()
vault.position()
vault.displayItem()   // предмет, который он тебе показывает, или null
vault.nbt()           // всё, что сервер про него прислал
```

`displayItem()` — это награда, крутящаяся внутри волта, и закопанная добыча внутри подозрительного песка или гравия:

```kotlin
on<ClientTickEvent> {
    whenInGame {
        val hit = raycast.crosshair(5.0)
        if (hit !is Hit.OnBlock) return@whenInGame
        val loot = world.block(hit.blockX(), hit.blockY(), hit.blockZ())
            .blockEntity()?.displayItem() ?: return@whenInGame
        chat.print("в этом волте ${loot.name()}")
    }
}
```

Это обычный [предмет](inventory.md), так что `id()`, `name()`, `isA("heavy_core")`, `enchantments()` и всё остальное на нём работает.

На всё прочее есть `nbt()` — сырые данные строкой, ровно как их пишет Minecraft:

```kotlin
val sign = world.block(position).blockEntity() ?: return
if (sign.type() == "minecraft:sign") {
    chat.print(sign.nbt())
}
```

## Найти блок-сущности вокруг

Вместо того чтобы самому обходить координаты, спрашивай сразу целый объём:

```kotlin
val vaults = world.blockEntitiesIn(Box.around(player.position(), 16.0))
    .filter { it.type() == "minecraft:vault" }

for (vault in vaults) {
    val loot = vault.displayItem() ?: continue
    if (loot.isA("heavy_core")) {
        chat.print("тяжёлое ядро на ${vault.x()} ${vault.y()} ${vault.z()}")
    }
}
```

Блок засчитывается, когда его центр внутри коробки. Коробка не может быть шире 32×32 чанков — это и так больше, чем у тебя вообще может быть прогружено.

Что учесть.

**Ты видишь только то, что сервер тебе прислал.** Крутящийся предмет волта приходит, потому что клиенту его надо нарисовать; содержимое сундука не приходит, пока ты сундук не откроешь, — так что блок-сущность сундука есть, но она пустая. Это правило самого Minecraft, а не ограничение API: в клиенте больше этого не знает никто.

**Оба вызова работают только на клиентском потоке.** Из обработчика пакетов оборачивай в `onClientThread { }`, иначе прилетит `ScriptThreadException`. Если тебе нужен только момент изменения блок-сущности, `S2CBlockEntityUpdatePacket` несёт те же данные в `nbt()` и его можно читать прямо там — см. [Пакеты](../actions/packets.md).

**Ничего из этого не достаёт до непрогруженных чанков.** За пределами дальности прорисовки блок-сущности просто нет.

## Жидкости

```kotlin
val fluid = world.block(position).fluid() ?: return

fluid.id()        // "minecraft:water"
fluid.level()     // 0..8, насколько блок заполнен
fluid.still()     // источник, а не течение
fluid.height()    // 0..1, докуда жидкость реально достаёт
```

У сухого блока `fluid()` — `null`. Вода внутри залитой водой лестницы считается: блок при этом `minecraft:oak_stairs`, а жидкость — вода.

Ещё пара полезных вещей про пространство:

```kotlin
world.lightLevel(x, y, z)      // освещённость
world.blockLight(x, y, z)      // только от факелов и ламп
world.skyLight(x, y, z)        // только от неба
world.biome(position)          // "minecraft:plains"
world.topY(x, z)               // высота самого верхнего блока
world.collisionsIn(box)        // все коллизии внутри коробки
world.isFree(box)              // пусто ли там
```

`lightLevel` — это то, что игра реально рисует. Разделяй, когда важно, *почему* в точке темно: `skyLight` под землёй равен нулю в любое время суток, а `blockLight` — везде, где никто не поставил факел.

## Коробки

`Box` — прямоугольный объём. Он есть у сущностей и блоков, и его можно собрать самому:

```kotlin
val around = Box.around(player.position(), 8.0)

around.center()
around.sizeX()
around.expand(1.0)             // раздуть во все стороны
around.contract(0.5)
around.offset(0.0, 1.0, 0.0)
around.contains(point)
around.intersects(other)
```

## Сущности вокруг

```kotlin
world.entities()                                    // все
world.entities(filters.player())                    // с фильтром
world.entitiesIn(Box.around(player.position(), 8.0))
world.entitiesNear(player.position(), 8.0, filters.attackable())
world.players()
world.entityCount()

world.entityById(id)
world.entityByUuid(uuid)
world.entityByName("Notch")
world.nearestEntity(player.position(), 16.0, filters.monster())
```

Всё, что возвращает одну сущность, может вернуть `null` — значит, не нашлось.

Про фильтры и что можно спросить у сущности — [Сущности и фильтры](entities.md).

## Удалить сущность

Сущность можно убрать из того мира, который видишь ты:

```kotlin
world.removeEntity(entity)       // по сущности
world.removeEntity(entityId)     // по id, если больше ничего нет
```

Возвращает `true`, когда что-то действительно убрали, и `false`, когда сущности уже не было — а также `false` на твоего собственного игрока, его не удалить.

Это только твоя копия мира и ничего больше. У сервера сущность на месте, он продолжает слать про неё обновления, а другие игроки видят её там же, где и раньше. Всё, что она с тобой делала — хитбокс на пути, взрыв, урон — продолжает происходить; ты лишь перестал её рисовать и вести локально. Пришлёт сервер её заново, на перезагрузке чанка или телепорте, — она вернётся.

```kotlin
on<ClientTickEvent> {
    whenInGame {
        for (entity in world.entities(filters.item())) {
            world.removeEntity(entity)
        }
    }
}
```

Что учесть. Работает только на клиентском потоке — из обработчика пакетов оборачивай в `onClientThread { }`, иначе прилетит `ScriptThreadException`. И удаление поднимает `EntityRemoveEvent`, в том числе твой собственный обработчик, так что удалять изнутри него стоит только осознанно.

## Вернуть спрятанные сущности

```kotlin
world.unhideEntities()
```

Снимает разом все `entity.hidden(true)` — см. [Как спрятать](entities.md#как-спрятать). Скрытие не принадлежит скрипту, который его поставил, поэтому этой строке место в твоём `onDisable`. С самими сущностями ничего не происходит: они никуда не девались, ты просто их не рисовал.

## Звуки и частицы

```kotlin
world.playSound("minecraft:entity.experience_orb.pickup", 1f, 1f)
world.playSound("minecraft:block.anvil.land", position, 1f, 0.8f)

world.spawnParticle("minecraft:flame", position, Vec.ZERO)
```

Без позиции звук играет там, где ты. Громкость и высота тона — обычные значения Minecraft: `1f` это нормально, `2f` выше, `0.5f` ниже.

Всё это видно и слышно **только тебе** — сервер об этом не знает.
