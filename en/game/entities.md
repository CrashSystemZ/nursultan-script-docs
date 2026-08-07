# Entities and filters

An entity is anything living in the world: players, mobs, boats, dropped items, arrows. You get them from the [world](world.md), from a [ray](raycast.md), or from an event.

## What any entity can tell you

```kotlin
entity.id()              // numeric id in this world
entity.uuid()
entity.name()
entity.typeId()          // "minecraft:zombie"

entity.position()
entity.renderPosition()  // smoothed, for drawing
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
entity.wet()             // in water or standing in the rain
entity.submerged()       // head under water
entity.inLava()
entity.frozen()  entity.frozenTicks()
entity.fireImmune()
entity.silent()
entity.noGravity()
entity.age()             // ticks since it appeared

entity.distanceTo(other)
entity.distanceTo(point)
```

## Names and teams

```kotlin
entity.name()            // plain text, always something
entity.displayName()     // Text: what the nametag actually draws
entity.hasCustomName()
entity.customName()      // the nametag someone set, or null

entity.team()            // scoreboard team name, or null
entity.teamColor()       // its colour as 0xAARRGGBB, or -1
```

On a server that puts players in coloured teams, `teamColor()` is how you tell friend from foe without a friends list:

```kotlin
val mine = player.teamColor()

val enemies = world.players().filter { it.teamColor() != mine }
```

`displayName()` is [styled text](../ui/text.md) — it carries the team prefix and colour the server sent.

## Riding

```kotlin
entity.vehicle()         // what it is riding, or null
entity.passengers()      // who is riding it
```

A player on a horse has the horse as their `vehicle()`, and hitting the player means aiming at the player, not the horse.

## Who it is

```kotlin
entity.isSelf()          // that is me
entity.isPlayer()
entity.isLiving()
entity.isBot()           // looks like a bot
entity.isFriend()        // on the friends list
entity.isParty()         // in my party
entity.isAlly()          // friend or party
```

`isAlly()` is usually the one you want so you do not hit your own people.

## Living entities

If an entity is alive it has health, effects and items. The cast returns `null` when it is, say, a boat:

```kotlin
val living = entity.asLiving() ?: return

living.health()
living.maxHealth()
living.absorption()
living.armorPoints()
living.bypassedHealth()   // health accounting for armor and resistance
living.dead()
living.hurtTicks()

living.blocking()         // holding a shield
living.usingItem()
living.isNaked()          // no armor

living.headYaw()
living.bodyYaw()

living.hasEffect("speed")
living.effect("speed")?.amplifier()
living.effects()

living.mainHandItem()
living.offHandItem()
living.activeItem()       // what it is using right now, or null
living.armorItem(ArmorSlot.HELMET)
living.armorItems()

living.baby()
living.scale()
living.climbing()
living.gliding()
living.sleeping()
living.usingRiptide()
living.itemUseTicks()     // for how long it has been using it
living.itemUseTicksLeft()
living.movementSpeed()
living.deathTicks()       // the death animation, 0 while alive
```

Attributes are the numbers behind all of that — speed, reach, knockback resistance, whatever the server has tweaked:

```kotlin
living.attribute("movement_speed")?.value()
living.attribute("minecraft:movement_speed")?.base()   // the namespace is optional

for ((id, attribute) in living.attributes()) {
    chat.print(id + " = " + attribute.value())
}
```

The ids: `movement_speed`, `attack_damage`, `max_health`, `block_interaction_range` and so on.

`base()` is the number before modifiers, `value()` is after them — the one that actually applies. `attributes()` walks every attribute the game knows, so call it once and keep the result rather than asking inside a loop.

Players have their own cast, which adds ping, game mode and the skin:

```kotlin
val target = entity.asPlayer() ?: return
target.pingMs()
target.gameMode()
target.skinTexture()         // the skin, or null
```

`skinTexture()` hands back a [texture](../ui/render-2d.md) you can draw pieces of or feed to a shader of your own — a head, a hat layer, anything cut out of the skin. It is `null` when the skin has not arrived yet and whenever Streamer Mode is hiding skins; draw nothing in that case rather than reaching for a fallback.

## Text displays

A text display is the other thing servers hang in the air. It is not an armor stand wearing a name — the label *is* the entity, so `hasCustomName()` is `false` and `customName()` is `null`. The text sits on its own cast:

```kotlin
val display = entity.asTextDisplay() ?: return

display.text()         // plain text
display.styledText()   // the same thing as styled text
```

`asTextDisplay()` is `null` for everything that is not one, exactly like `asLiving()`. A server may use either kind for the same job, so read whichever the entity actually has:

```kotlin
val label = when (entity.typeId()) {
    "minecraft:armor_stand" -> entity.customName()
    "minecraft:text_display" -> entity.asTextDisplay()?.text()
    else -> null
} ?: return@on
```

## Hiding one

You cannot stop the server from sending an entity, but you can stop your client from drawing it:

```kotlin
entity.hidden(true)
entity.hidden()          // is it hidden right now
```

That drops the nametag of any entity, and a display entity in full — a text display is nothing *but* its label, so nothing is left of it. Everything else carries on: the entity still ticks, its text still updates, and you can still read it. That is what separates this from `world.removeEntity(...)`, which throws your copy away and the updates with it.

Hiding lasts until you undo it or the chunk unloads, and it does not belong to your script — switch the script off and the entity stays hidden. Undo the lot in one call:

```kotlin
onDisable { world.unhideEntities() }
```

The vanilla invisibility flag is a separate thing, and the one `invisible()` reads:

```kotlin
entity.invisible(true)
```

It hides the *model* — an armor stand, a mob — and not the nametag above it. Like everything else here it is yours alone, so the server overwrites it the next time it sends that entity's metadata; set it again when you need it to hold. Display entities ignore the flag completely, which is why `hidden(...)` is the one that works on holograms.

## Filters

`filters` is a convenient way to pick entities by the traits you care about instead of writing the conditions out by hand:

| Filter | What it picks |
|---|---|
| `alive()` | living ones |
| `self()` | you |
| `player()` | players |
| `mob()` | mobs |
| `monster()` | hostile ones |
| `animal()` | animals |
| `villager()` | villagers |
| `item()` | dropped items |
| `friend()` | friends |
| `party()` | party members |
| `ally()` | friends and party |
| `bot()` | bots |
| `attackable()` | anything worth hitting |

Every filter is an ordinary `Predicate<Entity>`, so you can combine them:

```kotlin
val targets = world.entitiesNear(
    player.position(), 6.0,
    filters.player().and(filters.ally().negate())
)
```

And mix in your own conditions:

```kotlin
val weak = filters.attackable().and { it.asLiving()?.health() ?: 0f < 8f }

val victim = world.nearestEntity(player.position(), 5.0, weak)
```

## Spawning and despawning

```kotlin
on<EntitySpawnEvent> { e ->
    if (filters.monster().test(e.entity())) {
        notify("mob nearby", NotifyKind.WARN)
    }
}

on<EntityRemoveEvent> { e ->
    chat.print(e.entity().name() + " went away")
}
```

## Careful with holding on to them

Entities come and go. If you keep a reference across ticks, check it is still alive:

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

Keeping the `id()` and looking the entity up again with `world.entityById(...)` is safer.
