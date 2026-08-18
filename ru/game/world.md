# Мир и блоки

`world` — это `game.world()`. Любой вызов бросает `ScriptStateException`, когда мира нет; чтение блока — снимок на момент вызова.

```kotlin
on<ClientTickEvent> {
    whenInGame {
        val under = world.block(player.x().toInt(), player.y().toInt() - 1, player.z().toInt())
        val fluid = under.fluid()
        val nearby = world.entitiesNear(player.position(), 8.0, filters.attackable())
        chat.print("${under.id()} ${fluid?.id() ?: "сухо"} ${nearby.size}")
    }
}
```

## Сам мир

| Метод | Тип | Описание |
|---|---|---|
| `world.dimension()` | `String` | ключ измерения, например `minecraft:overworld` |
| `world.timeTicks()` | `long` | общий возраст мира в тиках |
| `world.dayTimeTicks()` | `long` | время суток в тиках, длина цикла 24000 |
| `world.raining()` | `boolean` | в этом мире идёт дождь |
| `world.thundering()` | `boolean` | в этом мире идёт гроза |
| `world.rainGradient()` | `float` | сила дождя 0..1 |
| `world.thunderGradient()` | `float` | сила грозы 0..1 |
| `world.biome(position)` | `String?` | id биома в позиции, округлённой вниз |
| `world.topY(x, z)` | `int` | y над верхним не-воздушным блоком, нижний y в непрогруженных |

## Прочитать блок

| Метод | Тип | Описание |
|---|---|---|
| `world.block(x, y, z)` | `Block` | снимок состояния блока, `void_air` за границами мира |
| `world.block(position)` | `Block` | то же, координаты `Vec` округляются вниз |
| `block.id()` | `String` | id блока, например `minecraft:stone` |
| `block.name()` | `String` | локализованное имя блока |
| `block.x()` | `int` | координата x блока |
| `block.y()` | `int` | координата y блока |
| `block.z()` | `int` | координата z блока |
| `block.position()` | `Vec` | координаты угла блока |
| `block.box()` | `Box` | куб 1×1×1 на этой позиции, не форма блока |

`Vec` и `Box` описаны на странице [Векторы, коробки, углы](math.md).

## Состояние блока

| Метод | Тип | Описание |
|---|---|---|
| `block.air()` | `boolean` | состояние — воздух |
| `block.solid()` | `boolean` | состояние твёрдое |
| `block.liquid()` | `boolean` | состояние — жидкий блок |
| `block.opaque()` | `boolean` | состояние непрозрачное |
| `block.blocksMovement()` | `boolean` | состояние блокирует движение |
| `block.replaceable()` | `boolean` | на место этого состояния можно поставить блок |
| `block.fullCube()` | `boolean` | состояние — полный куб на этой позиции |
| `block.hasCollision()` | `boolean` | форма коллизии на этой позиции не пустая |
| `block.luminance()` | `int` | излучаемый свет 0..15 |
| `block.opacity()` | `int` | поглощение света 0..15 |
| `block.hardness()` | `float` | прочность на слом здесь, -1 у неломаемого |
| `block.blastResistance()` | `float` | сопротивление взрыву |
| `block.slipperiness()` | `float` | трение, по умолчанию 0.6, у льда 0.98 |
| `block.velocityMultiplier()` | `float` | множитель горизонтальной скорости |
| `block.jumpVelocityMultiplier()` | `float` | множитель скорости прыжка |
| `block.emitsRedstone()` | `boolean` | состояние даёт редстоун-сигнал |
| `block.toolRequired()` | `boolean` | для дропа нужен подходящий инструмент |
| `block.burnable()` | `boolean` | состояние горючее |
| `block.randomTicks()` | `boolean` | состояние получает случайные тики |
| `block.pistonBehavior()` | `String` | строчными: normal, destroy, block, ignore, push_only |
| `block.mapColor()` | `int` | цвет на карте как ARGB, альфа принудительно 0xFF |

## Свойства и теги

| Метод | Тип | Описание |
|---|---|---|
| `block.properties()` | `Map<String, String>` | все свойства состояния как имя -> значение, неизменяемая, в порядке вставки |
| `block.property(name)` | `String?` | одно значение по имени без учёта регистра |
| `block.tags()` | `List<String>` | id тегов блока |
| `block.hasTag(tagId)` | `boolean` | блок в этом теге, ведущий `#` и неймспейс необязательны (бросает `ScriptException`, когда tagId пустой или не парсится) |

## Блок-сущности

| Метод | Тип | Описание |
|---|---|---|
| `block.hasBlockEntity()` | `boolean` | состояние объявляет блок-сущность |
| `block.blockEntity()` | `BlockEntity?` | блок-сущность на этой позиции (только главный поток) |
| `world.blockEntitiesIn(box)` | `List<BlockEntity>` | блок-сущности, чей центр блока внутри коробки, неизменяемый список (только главный поток) (бросает `ScriptException`, когда коробка шире 32×32 чанков) |
| `blockEntity.type()` | `String` | id типа блок-сущности, например `minecraft:vault` |
| `blockEntity.x()` | `int` | координата x блока |
| `blockEntity.y()` | `int` | координата y блока |
| `blockEntity.z()` | `int` | координата z блока |
| `blockEntity.position()` | `Vec` | координаты блока |
| `blockEntity.displayItem()` | `Item?` | предмет волта или добыча подозрительного блока, иначе null |
| `blockEntity.nbt()` | `String` | клиентский nbt строкой, пустая строка без мира |

Блок-сущность несёт только то, что уже прислал сервер: предмет волта есть, содержимое закрытого сундука — нет.
`displayItem()` возвращает обычный [предмет](inventory.md).

## Жидкости

| Метод | Тип | Описание |
|---|---|---|
| `block.fluid()` | `Fluid?` | состояние жидкости здесь, включая залитые водой блоки, null у сухого |
| `fluid.id()` | `String` | id жидкости, например `minecraft:water` |
| `fluid.level()` | `int` | уровень жидкости 1..8, 8 у источника |
| `fluid.still()` | `boolean` | источник, а не течение |
| `fluid.height()` | `float` | отрисованная высота жидкости 0..1 |

## Свет

| Метод | Тип | Описание |
|---|---|---|
| `world.lightLevel(x, y, z)` | `int` | отрисованная освещённость 0..15, с учётом темноты |
| `world.blockLight(x, y, z)` | `int` | свет от блоков 0..15 |
| `world.skyLight(x, y, z)` | `int` | свет от неба 0..15, без поправки на время суток |

## Сущности в мире

| Метод | Тип | Описание |
|---|---|---|
| `world.entityCount()` | `int` | количество прогруженных сущностей |
| `world.entities()` | `List<Entity>` | все прогруженные сущности, свой игрок включён, новый изменяемый список |
| `world.entities(filter)` | `List<Entity>` | сущности, прошедшие фильтр, null-фильтр пропускает всех |
| `world.entitiesIn(box)` | `List<Entity>` | сущности, чей хитбокс пересекает коробку |
| `world.entitiesIn(box, filter)` | `List<Entity>` | то же с фильтром, null-фильтр пропускает всех |
| `world.entitiesNear(origin, radiusBlocks, filter)` | `List<Entity>` | сущности в сфере радиусом в блоках (бросает `ScriptException`, когда radiusBlocks <= 0) |
| `world.players()` | `List<PlayerEntity>` | прогруженные игроки, неизменяемый список |
| `world.players(filter)` | `List<PlayerEntity>` | игроки, прошедшие фильтр, неизменяемый список |
| `world.entityByName(name)` | `Entity?` | первая сущность с таким отображаемым именем, без учёта регистра |
| `world.entityById(entityId)` | `Entity?` | сущность с таким сетевым id |
| `world.entityByUuid(uuid)` | `Entity?` | линейный поиск по uuid (бросает `ScriptException`, когда строка не uuid) |
| `world.nearestEntity(origin, radiusBlocks, filter)` | `Entity?` | ближайшая подходящая сущность в радиусе (бросает `ScriptException`, когда radiusBlocks <= 0) |

Что можно спросить у сущности и какие есть готовые фильтры — [Сущности и фильтры](entities.md).

## Столкновения

| Метод | Тип | Описание |
|---|---|---|
| `world.collisionsIn(box)` | `List<Box>` | коробки жёстко сталкивающихся сущностей внутри, коллизии блоков не входят |
| `world.isFree(box)` | `boolean` | ни одна сталкивающаяся сущность не пересекает коробку, блоки не проверяются |

## Удалить и спрятать

| Метод | Тип | Описание |
|---|---|---|
| `world.removeEntity(entity)` | `boolean` | клиентское удаление по обёртке, false при null (только главный поток) |
| `world.removeEntity(entityId)` | `boolean` | клиентское удаление по сетевому id, false для своего игрока (только главный поток) |
| `world.unhideEntities()` | `void` | снимает флаг [скрытия](entities.md#как-спрятать) со всех прогруженных сущностей |

Удаление только локальное: у сервера сущность остаётся и вернётся на перезагрузке чанка, а каждое удаление поднимает `EntityRemoveEvent`.

## Звуки и частицы

| Метод | Тип | Описание |
|---|---|---|
| `world.spawnParticle(particleId, position, velocity)` | `void` | частица только у тебя, null-скорость означает нулевую (бросает `ScriptException` на неизвестной частице) |
| `world.playSound(soundId, position, volume, pitch)` | `void` | звук только у тебя в точке, категория PLAYERS (бросает `ScriptException` на неизвестном звуке) |
| `world.playSound(soundId, volume, pitch)` | `void` | звук только у тебя у слушателя, категория PLAYERS (бросает `ScriptException` на неизвестном звуке) |
