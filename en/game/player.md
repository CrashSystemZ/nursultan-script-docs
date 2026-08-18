# Your player

`player` is `game.player()` — a live view of the local player, re-read on every call; every member throws `ScriptStateException` when there is no player. It is a `PlayerEntity`, so it carries everything an entity has: the tables below are the whole surface of `player`, inherited members included, and [Entities and filters](entities.md) documents those same members for any entity.

```kotlin
on<ClientTickEvent> {
    whenInGame {
        val eyes = player.eyePosition()
        val charge = player.attackCooldown()
        chat.print("hp ${player.health()} eyeY ${eyes.y()} charge $charge")
    }
}
```

## Position and movement

| Method | Type | Description |
|---|---|---|
| `player.position()` | [`Vec`](math.md#vec) | current feet position in world blocks |
| `player.x()` | `double` | current world X in blocks |
| `player.y()` | `double` | current world Y in blocks |
| `player.z()` | `double` | current world Z in blocks |
| `player.previousPosition()` | `Vec` | position at the previous tick |
| `player.renderPosition()` | `Vec` | tick-interpolated position used for drawing |
| `player.eyePosition()` | [`Vec`](math.md#vec) | eye position in world coordinates |
| `player.box()` | [`Box`](math.md#box) | bounding box in world coordinates |
| `player.width()` | `float` | hitbox width in blocks |
| `player.height()` | `float` | hitbox height in blocks |
| `player.rotation()` | [`Rotation`](math.md#rotation) | yaw and pitch in degrees |
| `player.yaw()` | `float` | yaw in degrees |
| `player.pitch()` | `float` | pitch in degrees, -90..90 |
| `player.bodyYaw()` | `float` | body yaw in degrees |
| `player.headYaw()` | `float` | head yaw in degrees |
| `player.velocity()` | `Vec` | velocity in blocks per tick |
| `player.movementSpeed()` | `float` | movement-speed attribute in blocks per tick |
| `player.distanceTo(other)` | `double` | distance between positions in blocks (throws `ScriptStateException` when `other` is not a world entity) |
| `player.distanceTo(point)` | `double` | distance from the position to a point in blocks |
| `player.fallDistanceBlocks()` | `double` | accumulated fall distance in blocks |

## State

| Method | Type | Description |
|---|---|---|
| `player.onGround()` | `boolean` | standing on ground |
| `player.sneaking()` | `boolean` | sneak flag |
| `player.sprinting()` | `boolean` | sprint flag |
| `player.wasSprinting()` | `boolean` | sprint state on the previous tick |
| `player.hasMovementInput()` | `boolean` | movement keys produce input this tick |
| `player.swimming()` | `boolean` | swimming flag |
| `player.crawling()` | `boolean` | crawling in a 1-block gap |
| `player.sleeping()` | `boolean` | entity is sleeping |
| `player.climbing()` | `boolean` | standing on a climbable block |
| `player.gliding()` | `boolean` | elytra gliding |
| `player.riding()` | `boolean` | has a vehicle |
| `player.flying()` | `boolean` | creative flight is active |
| `player.creative()` | `boolean` | creative abilities are set |
| `player.usingRiptide()` | `boolean` | in a riptide spin attack |
| `player.inWater()` | `boolean` | touching water |
| `player.inLava()` | `boolean` | standing in lava |
| `player.wet()` | `boolean` | touching water or standing in rain |
| `player.submerged()` | `boolean` | eyes are under water |
| `player.onFire()` | `boolean` | entity is burning |
| `player.fireImmune()` | `boolean` | entity type is immune to fire |
| `player.frozen()` | `boolean` | powder-snow freeze is fully applied |
| `player.frozenTicks()` | `int` | powder-snow freeze progress in ticks |
| `player.alive()` | `boolean` | alive and not removed from the world |
| `player.pose()` | [`Pose`](entities.md#poses) | current entity pose |
| `player.airTicks()` | `int` | remaining air in ticks |
| `player.maxAirTicks()` | `int` | maximum air in ticks |
| `player.baby()` | `boolean` | entity is a baby |
| `player.scale()` | `float` | scale multiplier, 1.0 by default |
| `player.silent()` | `boolean` | silent flag |
| `player.noGravity()` | `boolean` | no-gravity flag |
| `player.age()` | `int` | ticks the entity has existed on the client |
| `player.glowing()` | `boolean` | glowing flag |
| `player.invisible()` | `boolean` | vanilla invisibility flag |
| `player.invisible(value)` | `void` | sets it client-side; server metadata overwrites it |
| `player.hidden()` | `boolean` | client render-suppression flag; false for untracked entities |
| `player.hidden(value)` | `void` | sets the flag; no-op for untracked entities |
| `player.vehicle()` | [`Entity?`](entities.md) | entity being ridden, null when on foot |
| `player.passengers()` | [`List<Entity>`](entities.md) | entities riding this one, empty when none |

`invisible(true)` hides the model and not the nametag.
`hidden(true)` outlives the script being switched off — [`world.unhideEntities()`](world.md) clears it.

## Health and effects

| Method | Type | Description |
|---|---|---|
| `player.health()` | `float` | current health in half-hearts |
| `player.maxHealth()` | `float` | maximum health in half-hearts |
| `player.absorption()` | `float` | absorption (yellow) hearts |
| `player.armorPoints()` | `int` | armor points, 0..20 |
| `player.bypassedHealth()` | `float` | scoreboard health while BypassHealth is on, else `health()` |
| `player.dead()` | `boolean` | health is at or below zero |
| `player.deathTicks()` | `int` | ticks since death, 0 while alive |
| `player.hurtTicks()` | `int` | remaining hurt-animation ticks |
| `player.hasEffect(effectId)` | `boolean` | an effect with that exact namespaced id is active |
| `player.effects()` | [`List<Effect>`](entities.md#effects) | snapshots of every active status effect |
| `player.effect(effectId)` | [`Effect?`](entities.md#effects) | one active effect by exact namespaced id, null when absent |
| `player.attributes()` | [`Map<String, Attribute>`](entities.md#attributes) | unmodifiable map of every attribute present, keyed by namespaced id |
| `player.attribute(attributeId)` | [`Attribute?`](entities.md#attributes) | one attribute, `minecraft:` added when unqualified; null when absent |

`bypassedHealth()` equals `health()` in singleplayer and whenever the BypassHealth module is off.
Effect ids are compared exactly: `hasEffect("minecraft:speed")` matches, `hasEffect("speed")` does not.

## Hunger

| Method | Type | Description |
|---|---|---|
| `player.food()` | `int` | food level, 0..20 |
| `player.saturation()` | `float` | saturation level, 0..food |
| `player.hunger()` | `Hunger` | shared live hunger view (API 2) |

### The Hunger view

| Method | Type | Description |
|---|---|---|
| `hunger.food()` | `int` | food level, 0..20 |
| `hunger.maxFood()` | `int` | constant 20 |
| `hunger.saturation()` | `float` | saturation level, 0..food |
| `hunger.full()` | `boolean` | food level is at maximum |
| `hunger.canSprint()` | `boolean` | food level allows sprinting, above 6 |

Every `Hunger` member is API 2.
Exhaustion and the healing/starving tick counter are server-side, stay at zero on the client, and are not exposed.

## Experience

| Method | Type | Description |
|---|---|---|
| `player.xpLevel()` | `int` | experience level |
| `player.xpProgress()` | `float` | progress to the next level, 0..1 |

## Identity

| Method | Type | Description |
|---|---|---|
| `player.name()` | `String` | plain text of the entity name |
| `player.uuid()` | `String` | entity uuid in string form |
| `player.id()` | `int` | client network entity id |
| `player.typeId()` | `String` | namespaced entity-type id, e.g. `minecraft:player` |
| `player.displayName()` | [`Text`](../ui/text.md) | styled name, carries team colour and custom name |
| `player.hasCustomName()` | `boolean` | entity carries a custom name |
| `player.customName()` | `String?` | plain custom name, null when there is none |
| `player.team()` | `String?` | scoreboard team name, null when in no team |
| `player.teamColor()` | `int` | opaque ARGB team colour, -1 without team or colour |
| `player.isFriend()` | `boolean` | name is in the client friend list |
| `player.isParty()` | `boolean` | name is a party member |
| `player.isAlly()` | `boolean` | friend, party or NoFriendDamage teammate; always false while NoFriendDamage is off |
| `player.isBot()` | `boolean` | client bot heuristics; false for self and on FT servers |
| `player.pingMs()` | `int` | own player-list latency in milliseconds, 0 without an entry |
| `player.gameMode()` | [`GameMode`](entities.md#players) | current game mode, from the interaction manager |
| `player.skinTexture()` | [`Texture?`](../ui/render-2d.md) | body skin texture; null for non-client players and while StreamerMode hides skins |
| `player.isLiving()` | `boolean` | wrapped entity is a living entity |
| `player.isPlayer()` | `boolean` | wrapped entity is a player |
| `player.isSelf()` | `boolean` | wrapped entity is the local player |
| `player.asLiving()` | [`LivingEntity?`](entities.md#living-entities) | same entity as `LivingEntity`, null otherwise |
| `player.asPlayer()` | [`PlayerEntity?`](entities.md#players) | same entity as `PlayerEntity`, null otherwise |
| `player.asTextDisplay()` | [`TextDisplay?`](entities.md#text-displays) | same entity as `TextDisplay`, null otherwise |

## Equipment and item use

| Method | Type | Description |
|---|---|---|
| `player.mainHandItem()` | [`Item`](inventory.md) | main-hand stack, empty item when nothing is held |
| `player.offHandItem()` | `Item` | off-hand stack, empty item when nothing is held |
| `player.armorItem(slot)` | `Item` | stack in that [`ArmorSlot`](inventory.md), empty item when none |
| `player.armorItems()` | `List<Item>` | non-empty humanoid armor stacks only |
| `player.isNaked()` | `boolean` | no armor in any of the four humanoid slots |
| `player.activeItem()` | `Item?` | stack being used, null when none |
| `player.usingItem()` | `boolean` | currently using an item |
| `player.usingHand()` | [`Hand`](inventory.md#hands) | active hand, `MAIN_HAND` unless the off hand is active |
| `player.blocking()` | `boolean` | blocking with a shield |
| `player.itemUseTicks()` | `int` | ticks the current item has been used for |
| `player.itemUseTicksLeft()` | `int` | ticks left before the use completes |
| `player.swinging()` | `boolean` | hand-swing animation is running (API 2) |
| `player.swingTicks()` | `int` | current hand-swing animation tick (API 2) |

## Reach and attack cooldown

| Method | Type | Description |
|---|---|---|
| `player.entityReachBlocks()` | `double` | entity interaction range in blocks |
| `player.blockReachBlocks()` | `double` | block interaction range in blocks |
| `player.attackCooldown()` | `float` | attack charge at the tick boundary, 0..1 |
| `player.attackCooldown(tickDelta)` | `float` | attack charge interpolated by `tickDelta`, 0..1 (API 2) |
| `player.cooldownPeriod()` | `float` | full cooldown length in ticks for the held item (API 2) |
| `player.ticksSinceLastAttack()` | `int` | ticks since the last attack (API 2) |
| `player.belowMinimumAttackCharge()` | `boolean` | main-hand charge below the vanilla minimum, at `tickDelta` 0 |

## Actions

| Method | Type | Description |
|---|---|---|
| `player.respawn()` | `void` | sends a respawn request, queued onto the client thread |
