# Сущности и фильтры

Любая сущность, которую ты берёшь из [мира](world.md), из [луча](raycast.md) или из события, — живая обёртка: каждый геттер читает завёрнутую сущность Minecraft в момент вызова и продолжает читать её после того, как та ушла из мира, так что о пропаже говорит `alive()`. `LivingEntity`, `PlayerEntity`, `TextDisplay` и `ItemEntity` добавляют методы поверх `Entity`; локальный игрок — это [`SelfPlayer`](player.md).

```kotlin
on<ClientTickEvent> {
    val nearby = filters.player().and(filters.attackable())
    val target = world.nearestEntity(player.position(), 6.0, nearby)?.asLiving() ?: return@on

    chat.print("${target.name()} ${target.health()} / ${target.maxHealth()}")
    chat.print("box height ${target.box().sizeY()}")
}
```

## Кто это

| Метод | Тип | Описание |
|---|---|---|
| `entity.id()` | `int` | сетевой id сущности на клиенте |
| `entity.uuid()` | `String` | uuid сущности строкой |
| `entity.name()` | `String` | имя сущности обычным текстом |
| `entity.typeId()` | `String` | id типа с пространством имён, напр. `minecraft:zombie` |
| `entity.displayName()` | [`Text`](../ui/text.md) | оформленное имя, несёт цвет команды и кастомное имя |
| `entity.hasCustomName()` | `boolean` | на сущности висит кастомное имя |
| `entity.customName()` | `String?` | кастомное имя текстом, null если его нет |
| `entity.isLiving()` | `boolean` | завёрнутая сущность живая |
| `entity.isPlayer()` | `boolean` | завёрнутая сущность — игрок |
| `entity.isSelf()` | `boolean` | завёрнутая сущность — локальный игрок |
| `entity.isItem()` | `boolean` | завёрнутая сущность — выброшенный предмет (API 4) |
| `entity.asLiving()` | `LivingEntity?` | та же сущность как `LivingEntity`, иначе null |
| `entity.asPlayer()` | `PlayerEntity?` | та же сущность как `PlayerEntity`, иначе null |
| `entity.asTextDisplay()` | `TextDisplay?` | та же сущность как `TextDisplay`, иначе null |
| `entity.asItemEntity()` | `ItemEntity?` | та же сущность как `ItemEntity`, иначе null (API 4) |

## Позиция и размер

| Метод | Тип | Описание |
|---|---|---|
| `entity.position()` | [`Vec`](math.md#vec) | текущая позиция ног в блоках мира |
| `entity.x()` | `double` | текущий X в блоках |
| `entity.y()` | `double` | текущий Y в блоках |
| `entity.z()` | `double` | текущий Z в блоках |
| `entity.previousPosition()` | `Vec` | позиция на прошлом тике |
| `entity.serverPosition()` | `Vec` | позиция, которую последней прислал сервер, впереди клиентской интерполяции; `position()` для сущностей без интерполяции |
| `entity.interpolating()` | `boolean` | клиент всё ещё едет к `serverPosition()` |
| `entity.renderPosition()` | `Vec` | позиция с интерполяцией тика, для отрисовки |
| `entity.box()` | [`Box`](math.md#box) | хитбокс в координатах мира |
| `entity.width()` | `float` | ширина хитбокса в блоках |
| `entity.height()` | `float` | высота хитбокса в блоках |
| `entity.rotation()` | [`Rotation`](math.md#rotation) | yaw и pitch в градусах |
| `entity.yaw()` | `float` | yaw в градусах |
| `entity.yaw(value)` | `void` | ставит yaw только на клиенте (API 7) |
| `entity.pitch()` | `float` | pitch в градусах, -90..90 |
| `entity.pitch(value)` | `void` | ставит pitch только на клиенте (API 7) |
| `entity.velocity()` | `Vec` | скорость в блоках за тик |
| `entity.velocity(value)` | `void` | ставит скорость на клиенте; обновление с сервера перезапишет (API 7) |
| `entity.distanceTo(other)` | `double` | расстояние между позициями в блоках (бросает `ScriptStateException`, если `other` не сущность мира) |
| `entity.distanceTo(point)` | `double` | расстояние от позиции до точки в блоках |

## Состояние

| Метод | Тип | Описание |
|---|---|---|
| `entity.onGround()` | `boolean` | стоит на земле |
| `entity.sneaking()` | `boolean` | флаг приседания |
| `entity.sprinting()` | `boolean` | флаг спринта |
| `entity.swimming()` | `boolean` | флаг плавания |
| `entity.crawling()` | `boolean` | ползёт в щели высотой в один блок |
| `entity.wet()` | `boolean` | в воде или стоит под дождём |
| `entity.submerged()` | `boolean` | глаза под водой |
| `entity.inWater()` | `boolean` | касается воды |
| `entity.inLava()` | `boolean` | стоит в лаве |
| `entity.onFire()` | `boolean` | горит |
| `entity.fireImmune()` | `boolean` | тип сущности не боится огня |
| `entity.frozen()` | `boolean` | заморозка снегом применена полностью |
| `entity.frozenTicks()` | `int` | прогресс заморозки снегом в тиках |
| `entity.glowing()` | `boolean` | флаг свечения |
| `entity.invisible()` | `boolean` | ванильный флаг невидимости |
| `entity.invisible(value)` | `void` | ставит его только на клиенте; метаданные сервера перезапишут |
| `entity.alive()` | `boolean` | жива и не удалена из мира |
| `entity.pose()` | `Pose` | текущая поза |
| `entity.airTicks()` | `int` | остаток воздуха в тиках |
| `entity.maxAirTicks()` | `int` | максимум воздуха в тиках |
| `entity.fallDistanceBlocks()` | `double` | накопленная высота падения в блоках |
| `entity.fallDistanceBlocks(value)` | `void` | ставит накопленную высоту падения в блоках (API 7) |
| `entity.silent()` | `boolean` | флаг беззвучности |
| `entity.noGravity()` | `boolean` | флаг отсутствия гравитации |
| `entity.noClip()` | `boolean` | сущность проходит сквозь блоки (API 7) |
| `entity.noClip(value)` | `void` | ставит флаг прохождения сквозь блоки на клиенте (API 7) |
| `entity.age()` | `int` | сколько тиков сущность существует на клиенте |

`invisible(true)` прячет модель, но не неймтег, а дисплеи флаг игнорируют вовсе.

## Команды и отношения

| Метод | Тип | Описание |
|---|---|---|
| `entity.team()` | `String?` | имя команды на табло, null если команды нет |
| `entity.teamColor()` | `int` | непрозрачный ARGB цвет команды, -1 без команды или цвета |
| `entity.isFriend()` | `boolean` | имя есть в списке друзей клиента |
| `entity.isParty()` | `boolean` | имя есть в пати |
| `entity.isAlly()` | `boolean` | друг, пати или тиммейт NoFriendDamage; всегда false, пока NoFriendDamage выключен |
| `entity.isBot()` | `boolean` | эвристики клиента на бота; false для себя и на FT-серверах |

## Поездки

| Метод | Тип | Описание |
|---|---|---|
| `entity.vehicle()` | `Entity?` | на чём едет, null если едет сама |
| `entity.passengers()` | `List<Entity>` | кто едет на ней, пустой список если никто |

## Как спрятать

| Метод | Тип | Описание |
|---|---|---|
| `entity.hidden()` | `boolean` | клиентский флаг подавления отрисовки; false у неотслеживаемых сущностей |
| `entity.hidden(value)` | `void` | ставит флаг; ничего не делает у неотслеживаемых сущностей |

Скрытие убирает неймтег любой сущности, а текстовый дисплей — целиком; сама сущность тикает и остаётся читаемой.
Флаг переживает выключение скрипта — [`world.unhideEntities()`](world.md) снимает их все разом.

## Живые сущности

| Метод | Тип | Описание |
|---|---|---|
| `living.health()` | `float` | текущее здоровье в половинках сердец |
| `living.maxHealth()` | `float` | максимум здоровья в половинках сердец |
| `living.absorption()` | `float` | жёлтые сердца поглощения |
| `living.armorPoints()` | `int` | очки брони, 0..20 |
| `living.bypassedHealth()` | `float` | здоровье с табло, пока включён BypassHealth, иначе `health()` |
| `living.dead()` | `boolean` | здоровье не выше нуля |
| `living.deathTicks()` | `int` | тиков с момента смерти, 0 пока жива |
| `living.hurtTicks()` | `int` | остаток тиков анимации урона |
| `living.bodyYaw()` | `float` | yaw тела в градусах |
| `living.headYaw()` | `float` | yaw головы в градусах |
| `living.blocking()` | `boolean` | блокирует щитом |
| `living.usingItem()` | `boolean` | сейчас использует предмет |
| `living.activeItem()` | `Item?` | используемый стак, null если ничего |
| `living.itemUseTicks()` | `int` | сколько тиков использует предмет |
| `living.itemUseTicksLeft()` | `int` | сколько тиков осталось до конца использования |
| `living.swinging()` | `boolean` | идёт анимация замаха (API 2) |
| `living.swingTicks()` | `int` | текущий тик анимации замаха (API 2) |
| `living.baby()` | `boolean` | детёныш |
| `living.scale()` | `float` | множитель размера, по умолчанию 1.0 |
| `living.climbing()` | `boolean` | на блоке, по которому лезут |
| `living.gliding()` | `boolean` | летит на элитрах |
| `living.sleeping()` | `boolean` | спит |
| `living.usingRiptide()` | `boolean` | в риптайд-вращении трезубца |
| `living.movementSpeed()` | `float` | атрибут скорости в блоках за тик |
| `living.isNaked()` | `boolean` | ни в одном из четырёх слотов брони ничего нет |

`bypassedHealth()` равен `health()` в одиночной игре и всегда, когда BypassHealth выключен.

## Экипировка

| Метод | Тип | Описание |
|---|---|---|
| `living.mainHandItem()` | [`Item`](inventory.md) | стак в основной руке, пустой предмет если рука пуста |
| `living.offHandItem()` | `Item` | стак во второй руке, пустой предмет если она пуста |
| `living.armorItem(slot)` | `Item` | стак в этом [`ArmorSlot`](inventory.md), пустой предмет если слот пуст |
| `living.armorItems()` | `List<Item>` | только непустые стаки гуманоидной брони |

## Эффекты

| Метод | Тип | Описание |
|---|---|---|
| `living.hasEffect(effectId)` | `boolean` | активен эффект ровно с этим полным id |
| `living.effects()` | `List<Effect>` | снимки всех активных эффектов |
| `living.effect(effectId)` | `Effect?` | активный эффект по точному полному id, null если его нет |
| `living.visibleEffects()` | `List<String>` | id эффектов, снятые с частиц зелья, без уровня и длительности (API 3) |

Id эффектов сравниваются точно: `hasEffect("minecraft:speed")` попадает, `hasEffect("speed")` — нет.

### Effect

| Метод | Тип | Описание |
|---|---|---|
| `effect.id()` | `String` | id эффекта с пространством имён, напр. `minecraft:speed` |
| `effect.name()` | `String` | локализованное клиентом название эффекта |
| `effect.amplifier()` | `int` | уровень, 0 = I |
| `effect.durationTicks()` | `int` | остаток длительности в тиках |
| `effect.ambient()` | `boolean` | эффект от маяка или кондуита |
| `effect.infinite()` | `boolean` | длительность бесконечная |
| `effect.beneficial()` | `boolean` | тип эффекта считается полезным |

## Атрибуты

| Метод | Тип | Описание |
|---|---|---|
| `living.attributes()` | `Map<String, Attribute>` | неизменяемая карта всех имеющихся атрибутов по полному id |
| `living.attribute(attributeId)` | `Attribute?` | один атрибут, `minecraft:` добавится сам; null если атрибута нет |

`attributes()` на каждый вызов обходит весь реестр атрибутов.

### Attribute

| Метод | Тип | Описание |
|---|---|---|
| `attribute.id()` | `String` | id атрибута с пространством имён, напр. `minecraft:movement_speed` |
| `attribute.base()` | `double` | значение до модификаторов |
| `attribute.value()` | `double` | значение после всех модификаторов |

## Игроки

| Метод | Тип | Описание |
|---|---|---|
| `player.pingMs()` | `int` | пинг из списка игроков в миллисекундах, 0 без записи |
| `player.gameMode()` | `GameMode` | режим игры из списка игроков, `SURVIVAL` без записи |
| `player.skinTexture()` | [`Texture?`](../ui/render-2d.md) | текстура скина; null у неклиентских игроков и пока StreamerMode прячет скины |

### GameMode

| Константа | Описание |
|---|---|
| `SURVIVAL` | выживание; и запасной вариант, когда режим неизвестен |
| `CREATIVE` | творческий |
| `ADVENTURE` | приключение |
| `SPECTATOR` | наблюдатель |

## Текстовые дисплеи

| Метод | Тип | Описание |
|---|---|---|
| `display.text()` | `String` | текст дисплея без оформления |
| `display.styledText()` | `Text` | оформленный текст дисплея |

У текстового дисплея нет кастомного имени: `hasCustomName()` на нём false, а `customName()` — null.

### ItemEntity

| Метод | Тип | Описание |
|---|---|---|
| `itemEntity.stack()` | [`Item`](inventory.md) | стак, лежащий на земле, оборачивается на каждый вызов (API 4) |

Достаётся через `entity.asItemEntity()`; все методы `Entity` работают и на нём.

## Позы

| Константа | Описание |
|---|---|
| `STANDING` | обычная поза стоя |
| `GLIDING` | полёт на элитрах |
| `SLEEPING` | лежит в кровати |
| `SWIMMING` | плавание или ползание |
| `SPIN_ATTACK` | вращение с трезубцем |
| `CROUCHING` | приседание |
| `LONG_JUMPING` | длинный прыжок козы |
| `DYING` | анимация смерти |
| `CROAKING` | кваканье лягушки |
| `USING_TONGUE` | язык лягушки |
| `SITTING` | сидит, напр. верблюд |
| `ROARING` | рёв вардена |
| `SNIFFING` | принюхивание вардена или снифера |
| `EMERGING` | варден вылезает |
| `DIGGING` | варден закапывается |
| `SLIDING` | бриз скользит |
| `SHOOTING` | бриз стреляет |
| `INHALING` | бриз набирает воздух |

## Фильтры

| Метод | Тип | Описание |
|---|---|---|
| `filters.alive()` | `Predicate<Entity>` | сущность жива |
| `filters.self()` | `Predicate<Entity>` | сущность — локальный игрок |
| `filters.player()` | `Predicate<Entity>` | тип сущности `minecraft:player` |
| `filters.mob()` | `Predicate<Entity>` | сущность — моб |
| `filters.monster()` | `Predicate<Entity>` | сущность враждебная |
| `filters.animal()` | `Predicate<Entity>` | моб, который не враждебный |
| `filters.villager()` | `Predicate<Entity>` | тип сущности `minecraft:villager` |
| `filters.item()` | `Predicate<Entity>` | тип сущности `minecraft:item` |
| `filters.friend()` | `Predicate<Entity>` | сущность помечена как друг |
| `filters.party()` | `Predicate<Entity>` | сущность помечена как участник пати |
| `filters.ally()` | `Predicate<Entity>` | друг, пати или тиммейт NoFriendDamage; всегда false, пока NoFriendDamage выключен |
| `filters.bot()` | `Predicate<Entity>` | сущность помечена как бот |
| `filters.attackable()` | `Predicate<Entity>` | жива, не локальный игрок и не союзник |

Любой фильтр даёт false для объекта сущности, который скрипт получил не из мира.
