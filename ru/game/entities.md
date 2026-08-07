# Сущности и фильтры

Сущность — это всё, что живёт в мире: игроки, мобы, лодки, выпавшие предметы, стрелы. Берутся они из [мира](world.md), из [луча](raycast.md) или из события.

## Что можно спросить у любой сущности

```kotlin
entity.id()              // числовой id в этом мире
entity.uuid()
entity.name()
entity.typeId()          // "minecraft:zombie"

entity.position()
entity.renderPosition()  // сглаженная, для рисования
entity.previousPosition()
entity.box()
entity.width()  entity.height()

entity.rotation()
entity.yaw()  entity.pitch()
entity.velocity()

entity.alive()
entity.onGround()
entity.sneaking()
entity.sprinting()
entity.invisible()
entity.glowing()
entity.onFire()
entity.inWater()
entity.pose()
entity.fallDistanceBlocks()

entity.swimming()
entity.crawling()
entity.wet()             // в воде или стоит под дождём
entity.submerged()       // голова под водой
entity.inLava()
entity.frozen()  entity.frozenTicks()
entity.fireImmune()
entity.silent()
entity.noGravity()
entity.age()             // тиков с момента появления

entity.distanceTo(other)
entity.distanceTo(point)
```

## Имена и команды

```kotlin
entity.name()            // обычный текст, всегда что-то есть
entity.displayName()     // Text: то, что реально рисует нейметег
entity.hasCustomName()
entity.customName()      // имя, которое кто-то повесил, или null

entity.team()            // имя команды на табло, или null
entity.teamColor()       // её цвет как 0xAARRGGBB, или -1
```

На сервере, который раскидывает игроков по цветным командам, `teamColor()` — это способ отличить своего от чужого без списка друзей:

```kotlin
val mine = player.teamColor()

val enemies = world.players().filter { it.teamColor() != mine }
```

`displayName()` — это [оформленный текст](../ui/text.md): он несёт префикс и цвет команды, которые прислал сервер.

## Поездки

```kotlin
entity.vehicle()         // на чём едет, или null
entity.passengers()      // кто едет на нём
```

У игрока на лошади `vehicle()` — это лошадь, и бить игрока значит целиться в игрока, а не в лошадь.

## Кто это

```kotlin
entity.isSelf()          // это я
entity.isPlayer()
entity.isLiving()
entity.isBot()           // похоже на бота
entity.isFriend()        // в списке друзей
entity.isParty()         // в моей пати
entity.isAlly()          // друг или пати
```

`isAlly()` — то, что обычно и нужно, чтобы не бить своих.

## Живые сущности

Если сущность живая, у неё есть здоровье, эффекты и предметы. Приведение возвращает `null`, если это, например, лодка:

```kotlin
val living = entity.asLiving() ?: return

living.health()
living.maxHealth()
living.absorption()
living.armorPoints()
living.bypassedHealth()   // здоровье с учётом брони и резистов
living.dead()
living.hurtTicks()

living.blocking()         // держит щит
living.usingItem()
living.isNaked()          // без брони

living.headYaw()
living.bodyYaw()

living.hasEffect("speed")
living.effect("speed")?.amplifier()
living.effects()

living.mainHandItem()
living.offHandItem()
living.activeItem()       // что использует прямо сейчас, или null
living.armorItem(ArmorSlot.HELMET)
living.armorItems()

living.baby()
living.scale()
living.climbing()
living.gliding()
living.sleeping()
living.usingRiptide()
living.itemUseTicks()     // сколько уже использует
living.itemUseTicksLeft()
living.movementSpeed()
living.deathTicks()       // анимация смерти, 0 пока жив
```

Атрибуты — это числа за всем этим: скорость, дальность, сопротивление отбросу, что там сервер накрутил.

```kotlin
living.attribute("movement_speed")?.value()
living.attribute("minecraft:movement_speed")?.base()   // пространство имён необязательно

for ((id, attribute) in living.attributes()) {
    chat.print(id + " = " + attribute.value())
}
```

Идентификаторы: `movement_speed`, `attack_damage`, `max_health`, `block_interaction_range` и т.д.

`base()` — число до модификаторов, `value()` — после них, то есть то, которое реально действует. `attributes()` обходит все известные игре атрибуты, так что вызывай один раз и держи результат, а не спрашивай внутри цикла.

Для игроков есть отдельное приведение — оно даёт пинг, режим игры и скин:

```kotlin
val target = entity.asPlayer() ?: return
target.pingMs()
target.gameMode()
target.skinTexture()         // скин или null
```

`skinTexture()` отдаёт [текстуру](../ui/render-2d.md): из неё можно рисовать куски или скормить её своему шейдеру — голова, слой шапки, вообще что угодно, вырезанное из скина. Там `null`, пока скин не приехал, и всегда, когда режим стримера прячет скины: в этом случае лучше не рисовать ничего, чем подставлять заглушку.

## Текстовые дисплеи

Второе, что серверы вешают в воздух, — текстовый дисплей. Это не армор-стенд с именем: сама надпись и есть сущность, поэтому `hasCustomName()` у неё `false`, а `customName()` — `null`. Текст лежит за отдельным приведением:

```kotlin
val display = entity.asTextDisplay() ?: return

display.text()         // просто текст
display.styledText()   // то же самое как стилизованный текст
```

`asTextDisplay()` даёт `null` на всём, что дисплеем не является, — ровно как `asLiving()`. Под одну и ту же задачу сервер может взять любой из двух видов, так что читай то, что у сущности реально есть:

```kotlin
val label = when (entity.typeId()) {
    "minecraft:armor_stand" -> entity.customName()
    "minecraft:text_display" -> entity.asTextDisplay()?.text()
    else -> null
} ?: return@on
```

## Как спрятать

Запретить серверу присылать сущность нельзя, а вот запретить своему клиенту её рисовать — можно:

```kotlin
entity.hidden(true)
entity.hidden()          // спрятана ли она сейчас
```

Так пропадает неймтег любой сущности, а дисплей — целиком: текстовый дисплей и есть своя надпись, от него не остаётся ничего. Всё остальное продолжается: сущность тикает, её текст обновляется, и ты по-прежнему можешь его читать. Этим оно и отличается от `world.removeEntity(...)`, который выбрасывает твою копию, а вместе с ней и обновления.

Скрытие держится, пока ты его не снимешь или пока не выгрузится чанк, и оно не привязано к твоему скрипту — выключишь скрипт, а сущность останется спрятанной. Снять всё разом:

```kotlin
onDisable { world.unhideEntities() }
```

Ванильный флаг невидимости — отдельная вещь, и именно его читает `invisible()`:

```kotlin
entity.invisible(true)
```

Он прячет *модель* — армор-стенд, моба — но не неймтег над ней. Как и всё здесь, он твой личный, поэтому сервер перезапишет его при следующей отправке метаданных этой сущности: если нужно, чтобы держался, ставь заново. Дисплеи флаг игнорируют вовсе — потому на голограммах работает именно `hidden(...)`.

## Фильтры

`filters` даёт возможность удобно отбирать сущности по нужным признакам, не расписывая условия руками:

| Фильтр | Что отбирает |
|---|---|
| `alive()` | живые |
| `self()` | тебя |
| `player()` | игроков |
| `mob()` | мобов |
| `monster()` | враждебных |
| `animal()` | животных |
| `villager()` | жителей |
| `item()` | выпавшие предметы |
| `friend()` | друзей |
| `party()` | пати |
| `ally()` | друзей и пати |
| `bot()` | ботов |
| `attackable()` | всё, что имеет смысл бить |

Каждый фильтр — обычный `Predicate<Entity>`, так что их можно складывать:

```kotlin
val targets = world.entitiesNear(
    player.position(), 6.0,
    filters.player().and(filters.ally().negate())
)
```

И смешивать со своими условиями:

```kotlin
val weak = filters.attackable().and { it.asLiving()?.health() ?: 0f < 8f }

val victim = world.nearestEntity(player.position(), 5.0, weak)
```

## Появление и исчезновение

```kotlin
on<EntitySpawnEvent> { e ->
    if (filters.monster().test(e.entity())) {
        notify("рядом моб", NotifyKind.WARN)
    }
}

on<EntityRemoveEvent> { e ->
    chat.print(e.entity().name() + " пропал")
}
```

## Осторожно с хранением

Сущности приходят и уходят. Если держишь ссылку между тиками, проверяй, что она ещё жива:

```kotlin
var target: Entity? = null

on<ClientTickEvent> {
    val current = target
    if (current == null || !current.alive()) {
        target = world.nearestEntity(player.position(), 6.0, filters.attackable())
        return@on
    }
}
```

Надёжнее хранить `id()` и брать сущность заново через `world.entityById(...)`.
