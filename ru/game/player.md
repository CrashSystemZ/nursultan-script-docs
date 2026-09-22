# Свой игрок

`player` — это `game.player()`, живой вид на локального игрока: значение читается в момент вызова, а без игрока любой метод бросает `ScriptStateException`. Это `PlayerEntity`, поэтому у него есть всё, что есть у сущности: таблицы ниже — вся поверхность `player`, вместе с унаследованным, а [Сущности и фильтры](entities.md) описывают те же методы для любой сущности.

```kotlin
on<ClientTickEvent> {
    whenInGame {
        val eyes = player.eyePosition()
        val charge = player.attackCooldown()
        chat.print("хп ${player.health()} глаза ${eyes.y()} заряд $charge")
    }
}
```

## Позиция и движение

| Метод | Тип | Описание |
|---|---|---|
| `player.position()` | [`Vec`](math.md#vec) | текущая позиция ног в блоках мира |
| `player.x()` | `double` | текущий X в блоках |
| `player.y()` | `double` | текущий Y в блоках |
| `player.z()` | `double` | текущий Z в блоках |
| `player.previousPosition()` | `Vec` | позиция на прошлом тике |
| `player.renderPosition()` | `Vec` | позиция с интерполяцией тика, для отрисовки |
| `player.eyePosition()` | [`Vec`](math.md#vec) | позиция глаз в мировых координатах |
| `player.box()` | [`Box`](math.md#box) | хитбокс в координатах мира |
| `player.width()` | `float` | ширина хитбокса в блоках |
| `player.height()` | `float` | высота хитбокса в блоках |
| `player.rotation()` | [`Rotation`](math.md#rotation) | yaw и pitch в градусах |
| `player.yaw()` | `float` | yaw в градусах |
| `player.pitch()` | `float` | pitch в градусах, -90..90 |
| `player.bodyYaw()` | `float` | yaw тела в градусах |
| `player.headYaw()` | `float` | yaw головы в градусах |
| `player.velocity()` | `Vec` | скорость в блоках за тик |
| `player.velocity(value)` | `void` | перезаписывает её; гравитация и ввод применятся следующим тиком (API 5) (бросает `ScriptException`, если `value` null) |
| `player.movementSpeed()` | `float` | атрибут скорости в блоках за тик |
| `player.distanceTo(other)` | `double` | расстояние между позициями в блоках (бросает `ScriptStateException`, если `other` не сущность мира) |
| `player.distanceTo(point)` | `double` | расстояние от позиции до точки в блоках |
| `player.fallDistanceBlocks()` | `double` | накопленная высота падения в блоках |

## Состояние

| Метод | Тип | Описание |
|---|---|---|
| `player.onGround()` | `boolean` | стоит на земле |
| `player.sneaking()` | `boolean` | флаг приседания |
| `player.sprinting()` | `boolean` | флаг спринта |
| `player.serverSprinting()` | `boolean` | состояние спринта, о котором знает сервер (API 5) |
| `player.wasSprinting()` | `boolean` | старое имя `serverSprinting()`, то же значение |
| `player.hasMovementInput()` | `boolean` | клавиши движения дают ввод в этом тике |
| `player.swimming()` | `boolean` | флаг плавания |
| `player.crawling()` | `boolean` | ползёт в щели высотой в один блок |
| `player.sleeping()` | `boolean` | спит |
| `player.climbing()` | `boolean` | стоит на лазаемом блоке |
| `player.gliding()` | `boolean` | планирует на элитрах |
| `player.riding()` | `boolean` | сидит в транспорте |
| `player.flying()` | `boolean` | включён креативный полёт |
| `player.creative()` | `boolean` | выставлены креативные права |
| `player.usingRiptide()` | `boolean` | в риптайд-вращении трезубца |
| `player.inWater()` | `boolean` | касается воды |
| `player.inLava()` | `boolean` | стоит в лаве |
| `player.wet()` | `boolean` | в воде или стоит под дождём |
| `player.submerged()` | `boolean` | глаза под водой |
| `player.onFire()` | `boolean` | горит |
| `player.fireImmune()` | `boolean` | тип сущности не боится огня |
| `player.frozen()` | `boolean` | заморозка снегом применена полностью |
| `player.frozenTicks()` | `int` | прогресс заморозки снегом в тиках |
| `player.alive()` | `boolean` | жив и не удалён из мира |
| `player.pose()` | [`Pose`](entities.md#позы) | текущая поза |
| `player.airTicks()` | `int` | остаток воздуха в тиках |
| `player.maxAirTicks()` | `int` | максимум воздуха в тиках |
| `player.baby()` | `boolean` | детёныш |
| `player.scale()` | `float` | множитель размера, по умолчанию 1.0 |
| `player.silent()` | `boolean` | флаг беззвучности |
| `player.noGravity()` | `boolean` | флаг отсутствия гравитации |
| `player.age()` | `int` | сколько тиков сущность существует на клиенте |
| `player.glowing()` | `boolean` | флаг свечения |
| `player.invisible()` | `boolean` | ванильный флаг невидимости |
| `player.invisible(value)` | `void` | ставит его только на клиенте; метаданные сервера перезапишут |
| `player.hidden()` | `boolean` | клиентский флаг подавления отрисовки; false у неотслеживаемых сущностей |
| `player.hidden(value)` | `void` | ставит флаг; ничего не делает у неотслеживаемых сущностей |
| `player.vehicle()` | [`Entity?`](entities.md) | на чём едет, null если едет сам |
| `player.passengers()` | [`List<Entity>`](entities.md) | кто едет на нём, пустой список если никто |

`invisible(true)` прячет модель, но не неймтег.
`hidden(true)` переживает выключение скрипта — [`world.unhideEntities()`](world.md) снимает флаг.

## Здоровье и эффекты

| Метод | Тип | Описание |
|---|---|---|
| `player.health()` | `float` | текущее здоровье в половинках сердец |
| `player.maxHealth()` | `float` | максимум здоровья в половинках сердец |
| `player.absorption()` | `float` | жёлтые сердца поглощения |
| `player.armorPoints()` | `int` | очки брони, 0..20 |
| `player.bypassedHealth()` | `float` | здоровье с табло, пока включён BypassHealth, иначе `health()` |
| `player.dead()` | `boolean` | здоровье не выше нуля |
| `player.deathTicks()` | `int` | тиков с момента смерти, 0 пока жив |
| `player.hurtTicks()` | `int` | остаток тиков анимации урона |
| `player.hasEffect(effectId)` | `boolean` | активен эффект ровно с этим полным id |
| `player.effects()` | [`List<Effect>`](entities.md#эффекты) | снимки всех активных эффектов |
| `player.effect(effectId)` | [`Effect?`](entities.md#эффекты) | активный эффект по точному полному id, null если его нет |
| `player.attributes()` | [`Map<String, Attribute>`](entities.md#атрибуты) | неизменяемая карта всех имеющихся атрибутов по полному id |
| `player.attribute(attributeId)` | [`Attribute?`](entities.md#атрибуты) | один атрибут, `minecraft:` добавится сам; null если атрибута нет |

`bypassedHealth()` равен `health()` в одиночной игре и всегда, когда BypassHealth выключен.
Id эффектов сравниваются точно: `hasEffect("minecraft:speed")` попадает, `hasEffect("speed")` — нет.

## Голод

| Метод | Тип | Описание |
|---|---|---|
| `player.food()` | `int` | уровень голода, 0..20 |
| `player.saturation()` | `float` | сатурация, 0..food |
| `player.hunger()` | `Hunger` | общий живой вид на голод (API 2) |

### Объект Hunger

| Метод | Тип | Описание |
|---|---|---|
| `hunger.food()` | `int` | уровень голода, 0..20 |
| `hunger.maxFood()` | `int` | константа 20 |
| `hunger.saturation()` | `float` | сатурация, 0..food |
| `hunger.full()` | `boolean` | голод на максимуме |
| `hunger.canSprint()` | `boolean` | голода хватает на спринт, больше 6 |

Все методы `Hunger` — API 2.
Exhaustion и тиковый счётчик хила и урона от голода живут на сервере, на клиенте лежат нулями и не открыты.

## Опыт

| Метод | Тип | Описание |
|---|---|---|
| `player.xpLevel()` | `int` | уровень опыта |
| `player.xpProgress()` | `float` | прогресс до следующего уровня, 0..1 |

## Кто это

| Метод | Тип | Описание |
|---|---|---|
| `player.name()` | `String` | имя сущности обычным текстом |
| `player.uuid()` | `String` | uuid сущности строкой |
| `player.id()` | `int` | сетевой id сущности на клиенте |
| `player.typeId()` | `String` | id типа с пространством имён, напр. `minecraft:player` |
| `player.displayName()` | [`Text`](../ui/text.md) | оформленное имя, несёт цвет команды и кастомное имя |
| `player.hasCustomName()` | `boolean` | на сущности висит кастомное имя |
| `player.customName()` | `String?` | кастомное имя текстом, null если его нет |
| `player.team()` | `String?` | имя команды на табло, null если команды нет |
| `player.teamColor()` | `int` | непрозрачный ARGB цвет команды, -1 без команды или цвета |
| `player.isFriend()` | `boolean` | имя есть в списке друзей клиента |
| `player.isParty()` | `boolean` | имя есть в пати |
| `player.isAlly()` | `boolean` | друг, пати или тиммейт NoFriendDamage; всегда false, пока NoFriendDamage выключен |
| `player.isBot()` | `boolean` | эвристики клиента на бота; false для себя и на FT-серверах |
| `player.pingMs()` | `int` | свой пинг из списка игроков в миллисекундах, 0 без записи |
| `player.gameMode()` | [`GameMode`](entities.md#игроки) | текущий режим игры, из менеджера взаимодействия |
| `player.skinTexture()` | [`Texture?`](../ui/render-2d.md) | текстура скина; null у неклиентских игроков и пока StreamerMode прячет скины |
| `player.isLiving()` | `boolean` | завёрнутая сущность живая |
| `player.isPlayer()` | `boolean` | завёрнутая сущность — игрок |
| `player.isSelf()` | `boolean` | завёрнутая сущность — локальный игрок |
| `player.asLiving()` | [`LivingEntity?`](entities.md#живые-сущности) | та же сущность как `LivingEntity`, иначе null |
| `player.asPlayer()` | [`PlayerEntity?`](entities.md#игроки) | та же сущность как `PlayerEntity`, иначе null |
| `player.asTextDisplay()` | [`TextDisplay?`](entities.md#текстовые-дисплеи) | та же сущность как `TextDisplay`, иначе null |

## Экипировка и использование предмета

| Метод | Тип | Описание |
|---|---|---|
| `player.mainHandItem()` | [`Item`](inventory.md) | стак в основной руке, пустой предмет если рука пуста |
| `player.offHandItem()` | `Item` | стак во второй руке, пустой предмет если она пуста |
| `player.armorItem(slot)` | `Item` | стак в этом [`ArmorSlot`](inventory.md), пустой предмет если слот пуст |
| `player.armorItems()` | `List<Item>` | только непустые стаки гуманоидной брони |
| `player.isNaked()` | `boolean` | ни в одном из четырёх слотов брони ничего нет |
| `player.activeItem()` | `Item?` | используемый стак, null если ничего |
| `player.usingItem()` | `boolean` | сейчас использует предмет |
| `player.usingHand()` | [`Hand`](inventory.md#руки) | активная рука, `MAIN_HAND`, если не занята вторая |
| `player.blocking()` | `boolean` | блокирует щитом |
| `player.itemUseTicks()` | `int` | сколько тиков используется текущий предмет |
| `player.itemUseTicksLeft()` | `int` | сколько тиков осталось до конца использования |
| `player.swinging()` | `boolean` | идёт анимация замаха (API 2) |
| `player.swingTicks()` | `int` | текущий тик анимации замаха (API 2) |

## Досягаемость и откат атаки

| Метод | Тип | Описание |
|---|---|---|
| `player.entityReachBlocks()` | `double` | дальность взаимодействия с сущностями в блоках |
| `player.blockReachBlocks()` | `double` | дальность взаимодействия с блоками в блоках |
| `player.attackCooldown()` | `float` | заряд атаки на границе тика, 0..1 |
| `player.attackCooldown(tickDelta)` | `float` | заряд атаки, сглаженный по `tickDelta`, 0..1 (API 2) |
| `player.cooldownPeriod()` | `float` | полная длина отката в тиках для предмета в руке (API 2) |
| `player.ticksSinceLastAttack()` | `int` | сколько тиков прошло с последней атаки (API 2) |
| `player.belowMinimumAttackCharge()` | `boolean` | заряд основной руки ниже ванильного минимума, при `tickDelta` 0 |

## Действия

| Метод | Тип | Описание |
|---|---|---|
| `player.respawn()` | `void` | шлёт запрос на возрождение, ставится в очередь клиентского потока |
