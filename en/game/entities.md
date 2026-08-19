# Entities and filters

Every entity you get from [the world](world.md), a [ray](raycast.md) or an event is a live wrapper: each getter reads the wrapped Minecraft entity at call time, and it keeps reading it after the entity leaves the world, so `alive()` is what tells you it is gone. `LivingEntity`, `PlayerEntity` and `TextDisplay` add members on top of `Entity`; the local player is a [`SelfPlayer`](player.md).

```kotlin
on<ClientTickEvent> {
    val nearby = filters.player().and(filters.attackable())
    val target = world.nearestEntity(player.position(), 6.0, nearby)?.asLiving() ?: return@on

    chat.print("${target.name()} ${target.health()} / ${target.maxHealth()}")
    chat.print("box height ${target.box().sizeY()}")
}
```

## Identity

| Method | Type | Description |
|---|---|---|
| `entity.id()` | `int` | client network entity id |
| `entity.uuid()` | `String` | entity uuid in string form |
| `entity.name()` | `String` | plain text of the entity name |
| `entity.typeId()` | `String` | namespaced entity-type id, e.g. `minecraft:zombie` |
| `entity.displayName()` | [`Text`](../ui/text.md) | styled name, carries team colour and custom name |
| `entity.hasCustomName()` | `boolean` | entity carries a custom name |
| `entity.customName()` | `String?` | plain custom name, null when there is none |
| `entity.isLiving()` | `boolean` | wrapped entity is a living entity |
| `entity.isPlayer()` | `boolean` | wrapped entity is a player |
| `entity.isSelf()` | `boolean` | wrapped entity is the local player |
| `entity.asLiving()` | `LivingEntity?` | same entity as `LivingEntity`, null otherwise |
| `entity.asPlayer()` | `PlayerEntity?` | same entity as `PlayerEntity`, null otherwise |
| `entity.asTextDisplay()` | `TextDisplay?` | same entity as `TextDisplay`, null otherwise |

## Position and size

| Method | Type | Description |
|---|---|---|
| `entity.position()` | [`Vec`](math.md#vec) | current feet position in world blocks |
| `entity.x()` | `double` | current world X in blocks |
| `entity.y()` | `double` | current world Y in blocks |
| `entity.z()` | `double` | current world Z in blocks |
| `entity.previousPosition()` | `Vec` | position at the previous tick |
| `entity.renderPosition()` | `Vec` | tick-interpolated position used for drawing |
| `entity.box()` | [`Box`](math.md#box) | bounding box in world coordinates |
| `entity.width()` | `float` | hitbox width in blocks |
| `entity.height()` | `float` | hitbox height in blocks |
| `entity.rotation()` | [`Rotation`](math.md#rotation) | yaw and pitch in degrees |
| `entity.yaw()` | `float` | yaw in degrees |
| `entity.pitch()` | `float` | pitch in degrees, -90..90 |
| `entity.velocity()` | `Vec` | velocity in blocks per tick |
| `entity.distanceTo(other)` | `double` | distance between positions in blocks (throws `ScriptStateException` when `other` is not a world entity) |
| `entity.distanceTo(point)` | `double` | distance from the position to a point in blocks |

## State

| Method | Type | Description |
|---|---|---|
| `entity.onGround()` | `boolean` | standing on ground |
| `entity.sneaking()` | `boolean` | sneak flag |
| `entity.sprinting()` | `boolean` | sprint flag |
| `entity.swimming()` | `boolean` | swimming flag |
| `entity.crawling()` | `boolean` | crawling in a 1-block gap |
| `entity.wet()` | `boolean` | touching water or standing in rain |
| `entity.submerged()` | `boolean` | eyes are under water |
| `entity.inWater()` | `boolean` | touching water |
| `entity.inLava()` | `boolean` | standing in lava |
| `entity.onFire()` | `boolean` | entity is burning |
| `entity.fireImmune()` | `boolean` | entity type is immune to fire |
| `entity.frozen()` | `boolean` | powder-snow freeze is fully applied |
| `entity.frozenTicks()` | `int` | powder-snow freeze progress in ticks |
| `entity.glowing()` | `boolean` | glowing flag |
| `entity.invisible()` | `boolean` | vanilla invisibility flag |
| `entity.invisible(value)` | `void` | sets it client-side; server metadata overwrites it |
| `entity.alive()` | `boolean` | alive and not removed from the world |
| `entity.pose()` | `Pose` | current entity pose |
| `entity.airTicks()` | `int` | remaining air in ticks |
| `entity.maxAirTicks()` | `int` | maximum air in ticks |
| `entity.fallDistanceBlocks()` | `double` | accumulated fall distance in blocks |
| `entity.silent()` | `boolean` | silent flag |
| `entity.noGravity()` | `boolean` | no-gravity flag |
| `entity.age()` | `int` | ticks the entity has existed on the client |

`invisible(true)` hides the model and not the nametag, and display entities ignore the flag entirely.

## Teams and relations

| Method | Type | Description |
|---|---|---|
| `entity.team()` | `String?` | scoreboard team name, null when in no team |
| `entity.teamColor()` | `int` | opaque ARGB team colour, -1 without team or colour |
| `entity.isFriend()` | `boolean` | name is in the client friend list |
| `entity.isParty()` | `boolean` | name is a party member |
| `entity.isAlly()` | `boolean` | friend, party or NoFriendDamage teammate; always false while NoFriendDamage is off |
| `entity.isBot()` | `boolean` | client bot heuristics; false for self and on FT servers |

## Riding

| Method | Type | Description |
|---|---|---|
| `entity.vehicle()` | `Entity?` | entity being ridden, null when on foot |
| `entity.passengers()` | `List<Entity>` | entities riding this one, empty when none |

## Hiding one

| Method | Type | Description |
|---|---|---|
| `entity.hidden()` | `boolean` | client render-suppression flag; false for untracked entities |
| `entity.hidden(value)` | `void` | sets the flag; no-op for untracked entities |

Hiding drops the nametag of any entity and a text display in full; the entity keeps ticking and stays readable.
The flag outlives the script being switched off — [`world.unhideEntities()`](world.md) clears every one of them.

## Living entities

| Method | Type | Description |
|---|---|---|
| `living.health()` | `float` | current health in half-hearts |
| `living.maxHealth()` | `float` | maximum health in half-hearts |
| `living.absorption()` | `float` | absorption (yellow) hearts |
| `living.armorPoints()` | `int` | armor points, 0..20 |
| `living.bypassedHealth()` | `float` | scoreboard health while BypassHealth is on, else `health()` |
| `living.dead()` | `boolean` | health is at or below zero |
| `living.deathTicks()` | `int` | ticks since death, 0 while alive |
| `living.hurtTicks()` | `int` | remaining hurt-animation ticks |
| `living.bodyYaw()` | `float` | body yaw in degrees |
| `living.headYaw()` | `float` | head yaw in degrees |
| `living.blocking()` | `boolean` | blocking with a shield |
| `living.usingItem()` | `boolean` | currently using an item |
| `living.activeItem()` | `Item?` | stack being used, null when none |
| `living.itemUseTicks()` | `int` | ticks the current item has been used for |
| `living.itemUseTicksLeft()` | `int` | ticks left before the use completes |
| `living.swinging()` | `boolean` | hand-swing animation is running (API 2) |
| `living.swingTicks()` | `int` | current hand-swing animation tick (API 2) |
| `living.baby()` | `boolean` | entity is a baby |
| `living.scale()` | `float` | scale multiplier, 1.0 by default |
| `living.climbing()` | `boolean` | on a climbable block |
| `living.gliding()` | `boolean` | elytra-gliding |
| `living.sleeping()` | `boolean` | entity is sleeping |
| `living.usingRiptide()` | `boolean` | in a riptide spin attack |
| `living.movementSpeed()` | `float` | movement-speed attribute in blocks per tick |
| `living.isNaked()` | `boolean` | no armor in any of the four humanoid slots |

`bypassedHealth()` equals `health()` in singleplayer and whenever the BypassHealth module is off.

## Equipment

| Method | Type | Description |
|---|---|---|
| `living.mainHandItem()` | [`Item`](inventory.md) | main-hand stack, empty item when nothing is held |
| `living.offHandItem()` | `Item` | off-hand stack, empty item when nothing is held |
| `living.armorItem(slot)` | `Item` | stack in that [`ArmorSlot`](inventory.md), empty item when none |
| `living.armorItems()` | `List<Item>` | non-empty humanoid armor stacks only |

## Effects

| Method | Type | Description |
|---|---|---|
| `living.hasEffect(effectId)` | `boolean` | an effect with that exact namespaced id is active |
| `living.effects()` | `List<Effect>` | snapshots of every active status effect |
| `living.effect(effectId)` | `Effect?` | one active effect by exact namespaced id, null when absent |
| `living.visibleEffects()` | `List<String>` | effect ids read off the potion particles, no amplifier or duration (API 3) |

Effect ids are compared exactly: `hasEffect("minecraft:speed")` matches, `hasEffect("speed")` does not.

### Effect

| Method | Type | Description |
|---|---|---|
| `effect.id()` | `String` | namespaced effect id, e.g. `minecraft:speed` |
| `effect.name()` | `String` | client-localized effect name |
| `effect.amplifier()` | `int` | amplifier, 0 = level I |
| `effect.durationTicks()` | `int` | remaining duration in ticks |
| `effect.ambient()` | `boolean` | effect came from a beacon or conduit |
| `effect.infinite()` | `boolean` | effect has infinite duration |
| `effect.beneficial()` | `boolean` | effect type is classed as beneficial |

## Attributes

| Method | Type | Description |
|---|---|---|
| `living.attributes()` | `Map<String, Attribute>` | unmodifiable map of every attribute present, keyed by namespaced id |
| `living.attribute(attributeId)` | `Attribute?` | one attribute, `minecraft:` added when unqualified; null when absent |

`attributes()` walks the whole attribute registry on each call.

### Attribute

| Method | Type | Description |
|---|---|---|
| `attribute.id()` | `String` | namespaced attribute id, e.g. `minecraft:movement_speed` |
| `attribute.base()` | `double` | value before modifiers |
| `attribute.value()` | `double` | value after every modifier |

## Players

| Method | Type | Description |
|---|---|---|
| `player.pingMs()` | `int` | player-list latency in milliseconds, 0 without an entry |
| `player.gameMode()` | `GameMode` | player-list game mode, `SURVIVAL` without an entry |
| `player.skinTexture()` | [`Texture?`](../ui/render-2d.md) | body skin texture; null for non-client players and while StreamerMode hides skins |

### GameMode

| Constant | Description |
|---|---|
| `SURVIVAL` | survival mode; also the fallback when the mode is unknown |
| `CREATIVE` | creative mode |
| `ADVENTURE` | adventure mode |
| `SPECTATOR` | spectator mode |

## Text displays

| Method | Type | Description |
|---|---|---|
| `display.text()` | `String` | plain text of the display |
| `display.styledText()` | `Text` | styled text of the display |

A text display has no custom name: `hasCustomName()` is false and `customName()` is null on it.

## Poses

| Constant | Description |
|---|---|
| `STANDING` | default upright pose |
| `GLIDING` | elytra flight |
| `SLEEPING` | lying in a bed |
| `SWIMMING` | swimming or crawling posture |
| `SPIN_ATTACK` | trident riptide spin |
| `CROUCHING` | sneaking |
| `LONG_JUMPING` | goat long jump |
| `DYING` | death animation |
| `CROAKING` | frog croak |
| `USING_TONGUE` | frog tongue |
| `SITTING` | sitting, camel and the like |
| `ROARING` | warden roar |
| `SNIFFING` | warden or sniffer sniff |
| `EMERGING` | warden emerging |
| `DIGGING` | warden digging |
| `SLIDING` | breeze slide |
| `SHOOTING` | breeze shooting |
| `INHALING` | breeze inhaling |

## Filters

| Method | Type | Description |
|---|---|---|
| `filters.alive()` | `Predicate<Entity>` | entity is alive |
| `filters.self()` | `Predicate<Entity>` | entity is the local player |
| `filters.player()` | `Predicate<Entity>` | entity type is `minecraft:player` |
| `filters.mob()` | `Predicate<Entity>` | entity is a mob |
| `filters.monster()` | `Predicate<Entity>` | entity is hostile |
| `filters.animal()` | `Predicate<Entity>` | mob that is not hostile |
| `filters.villager()` | `Predicate<Entity>` | entity type is `minecraft:villager` |
| `filters.item()` | `Predicate<Entity>` | entity type is `minecraft:item` |
| `filters.friend()` | `Predicate<Entity>` | entity is flagged as a friend |
| `filters.party()` | `Predicate<Entity>` | entity is flagged as a party member |
| `filters.ally()` | `Predicate<Entity>` | friend, party or NoFriendDamage teammate; always false while NoFriendDamage is off |
| `filters.bot()` | `Predicate<Entity>` | entity is flagged as a bot |
| `filters.attackable()` | `Predicate<Entity>` | alive, not the local player and not an ally |

Every filter returns false for an entity object the script did not get from the world.
