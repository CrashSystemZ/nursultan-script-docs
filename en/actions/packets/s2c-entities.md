# Entity packets

Server-to-client records about entities and the local player: spawning, movement, rotation, velocity, damage, effects, equipment, attributes and tracked data. Positions are world blocks, velocities blocks per tick, rotations degrees and durations ticks; threading, cancelling and what the decode drops are on [Packets](../packets.md).

```kotlin
on<PacketReceiveEvent> { e ->
    val packet = e.packet()
    if (packet !is S2CEntityVelocityPacket) return@on
    if (packet.entityId() != player.id()) return@on

    log.info("knockback ${packet.velX()} ${packet.velZ()}")
}
```

## S2CDamageTiltPacket

Hurt-direction camera tilt.

| Component | Type | Description |
|---|---|---|
| `id()` | `int` | damaged entity network id |
| `yaw()` | `float` | direction the damage came from, degrees |

## S2CDeathMessagePacket

Death screen message.

| Component | Type | Description |
|---|---|---|
| `playerId()` | `int` | dead player entity network id |
| `message()` | `String` | plain-text death message |

## S2CEndCombatPacket

Combat tracker leaves combat. No components.

## S2CEnterCombatPacket

Combat tracker enters combat. No components.

## S2CEntitiesDestroyPacket

Entities removed from the client world.

| Component | Type | Description |
|---|---|---|
| `entityIds()` | `List<Integer>` | network ids of the removed entities |

## S2CEntityAnimationPacket

Entity animation trigger.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | animating entity network id |
| `animationId()` | `int` | 0 swing main hand, 2 wake up, 3 swing off hand, 4 crit, 5 enchanted hit |

## S2CEntityAttachPacket

Leash attached or detached.

| Component | Type | Description |
|---|---|---|
| `attachedEntityId()` | `int` | leashed entity network id |
| `holdingEntityId()` | `int` | holder entity network id, -1 detaches |

## S2CEntityAttributesPacket

Attribute base values and modifiers for one entity.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | entity network id |
| `attributes()` | `List<Attribute>` | attribute entries |

### S2CEntityAttributesPacket.Attribute

| Component | Type | Description |
|---|---|---|
| `attribute()` | `String` | attribute id with namespace, e.g. `minecraft:movement_speed` |
| `base()` | `double` | base value before modifiers |
| `modifiers()` | `List<Modifier>` | modifiers applied to this attribute |

### S2CEntityAttributesPacket.Modifier

| Component | Type | Description |
|---|---|---|
| `id()` | `String` | modifier id with namespace |
| `value()` | `double` | modifier amount |
| `operation()` | `String` | lowercased vanilla operation, e.g. `add_value`, `add_multiplied_base` |

## S2CEntityDamagePacket

Damage taken by an entity. The damage type and the amount are not exposed.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | damaged entity network id |
| `sourceCauseId()` | `int` | attacker entity network id, 0 when none |
| `sourceDirectId()` | `int` | direct source network id, projectile included, 0 when none |
| `sourceX()` | `Double?` | damage source world x, null for most sources |
| `sourceY()` | `Double?` | damage source world y, null for most sources |
| `sourceZ()` | `Double?` | damage source world z, null for most sources |

## S2CEntityEquipmentUpdatePacket

Equipment slot contents for one entity.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | entity network id |
| `equipment()` | `List<Equipment>` | changed equipment slots |

### S2CEntityEquipmentUpdatePacket.Equipment

| Component | Type | Description |
|---|---|---|
| `slot()` | `String` | vanilla slot name: `mainhand`, `offhand`, `head`, `chest`, `legs`, `feet`, `body`, `saddle` |
| `item()` | `String` | item id with namespace, `minecraft:air` when empty |
| `count()` | `int` | stack size |

## S2CEntityMoveRelativePacket

Relative entity move without rotation.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | entity network id |
| `deltaX()` | `short` | x delta in 1/4096 block units |
| `deltaY()` | `short` | y delta in 1/4096 block units |
| `deltaZ()` | `short` | z delta in 1/4096 block units |
| `onGround()` | `boolean` | entity on ground flag |

## S2CEntityPassengersSetPacket

Passenger list of a vehicle entity.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | vehicle entity network id |
| `passengerIds()` | `List<Integer>` | passenger network ids, empty dismounts everyone |

## S2CEntityPositionPacket

Absolute entity teleport with velocity and rotation.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | entity network id |
| `x()` | `double` | x in blocks |
| `y()` | `double` | y in blocks |
| `z()` | `double` | z in blocks |
| `velX()` | `double` | x velocity in blocks per tick |
| `velY()` | `double` | y velocity in blocks per tick |
| `velZ()` | `double` | z velocity in blocks per tick |
| `yaw()` | `float` | yaw in degrees |
| `pitch()` | `float` | pitch in degrees |
| `relatives()` | `List<PositionFlag>` | fields that are deltas on the current values — [Packet enums](enums.md) |
| `onGround()` | `boolean` | entity on ground flag |

## S2CEntityPositionSyncPacket

Periodic absolute position resync for one entity.

| Component | Type | Description |
|---|---|---|
| `id()` | `int` | entity network id |
| `x()` | `double` | x in blocks |
| `y()` | `double` | y in blocks |
| `z()` | `double` | z in blocks |
| `velX()` | `double` | x velocity in blocks per tick |
| `velY()` | `double` | y velocity in blocks per tick |
| `velZ()` | `double` | z velocity in blocks per tick |
| `yaw()` | `float` | yaw in degrees |
| `pitch()` | `float` | pitch in degrees |
| `onGround()` | `boolean` | entity on ground flag |

## S2CEntityRotateAndMoveRelativePacket

Relative entity move plus rotation.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | entity network id |
| `deltaX()` | `short` | x delta in 1/4096 block units |
| `deltaY()` | `short` | y delta in 1/4096 block units |
| `deltaZ()` | `short` | z delta in 1/4096 block units |
| `yaw()` | `float` | yaw in degrees, quantised to 360/256 steps on the wire |
| `pitch()` | `float` | pitch in degrees, quantised to 360/256 steps on the wire |
| `onGround()` | `boolean` | entity on ground flag |

## S2CEntityRotatePacket

Entity rotation only. The entity id is not exposed.

| Component | Type | Description |
|---|---|---|
| `yaw()` | `float` | yaw in degrees, quantised to 360/256 steps on the wire |
| `pitch()` | `float` | pitch in degrees, quantised to 360/256 steps on the wire |
| `onGround()` | `boolean` | entity on ground flag |

## S2CEntitySetHeadYawPacket

Entity head yaw. The entity id is not exposed.

| Component | Type | Description |
|---|---|---|
| `headYaw()` | `float` | head yaw in degrees, quantised to 360/256 steps on the wire |

## S2CEntitySpawnPacket

Entity added to the client world.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | entity network id |
| `uuid()` | `String` | entity uuid string |
| `entityType()` | `String` | entity type id with namespace |
| `x()` | `double` | spawn x in blocks |
| `y()` | `double` | spawn y in blocks |
| `z()` | `double` | spawn z in blocks |
| `velX()` | `double` | x velocity in blocks per tick |
| `velY()` | `double` | y velocity in blocks per tick |
| `velZ()` | `double` | z velocity in blocks per tick |
| `pitch()` | `float` | pitch in degrees |
| `yaw()` | `float` | yaw in degrees |
| `headYaw()` | `float` | head yaw in degrees |
| `entityData()` | `int` | type-specific spawn data: owner id, block state id, orientation |

## S2CEntityStatusEffectPacket

Status effect applied to or refreshed on an entity.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | affected entity network id |
| `effect()` | `String` | effect id with namespace |
| `amplifier()` | `int` | 0-based level, 0 is level I |
| `duration()` | `int` | remaining duration in ticks, -1 is infinite |
| `ambient()` | `boolean` | comes from a beacon or conduit |
| `showParticles()` | `boolean` | effect particles render |
| `showIcon()` | `boolean` | HUD icon shows |
| `keepFading()` | `boolean` | blend animation keeps running |

## S2CEntityStatusPacket

Entity event code: crit particles, death, taming, totem pop. The entity id is not exposed.

| Component | Type | Description |
|---|---|---|
| `status()` | `byte` | vanilla entity status code |

## S2CEntityTrackerUpdatePacket

Tracked data update for one entity.

| Component | Type | Description |
|---|---|---|
| `id()` | `int` | entity network id |
| `values()` | `List<TrackedValue>` | changed entries, empty when the packet carries none |

### S2CEntityTrackerUpdatePacket.TrackedValue

| Component | Type | Description |
|---|---|---|
| `id()` | `int` | data tracker entry index |
| `value()` | `String` | `String.valueOf` of the raw object, format follows its type |

## S2CEntityValueDebugPacket

Debug entity-value push. The value payload is not exposed.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | entity network id |

## S2CEntityVelocityPacket

Entity velocity set: knockback, explosion, fluid push.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | entity network id |
| `velX()` | `double` | x velocity in blocks per tick |
| `velY()` | `double` | y velocity in blocks per tick |
| `velZ()` | `double` | z velocity in blocks per tick |

## S2CExperienceBarUpdatePacket

Experience bar, level and total points.

| Component | Type | Description |
|---|---|---|
| `barProgress()` | `float` | bar fill, 0..1 |
| `experienceLevel()` | `int` | current level |
| `experience()` | `int` | total experience points |

## S2CHealthUpdatePacket

Local player health, food and saturation.

| Component | Type | Description |
|---|---|---|
| `health()` | `float` | health points, 20 is full for an unmodified player |
| `food()` | `int` | food level, 0..20 |
| `saturation()` | `float` | saturation, 0..20 |

## S2CItemPickupAnimationPacket

Item flying towards the entity that collected it.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | picked-up item entity network id |
| `collectorEntityId()` | `int` | collecting entity network id |
| `stackAmount()` | `int` | number of items collected |

## S2CLookAtPacket

Server forces the player to face a target. The target itself is not exposed.

| Component | Type | Description |
|---|---|---|
| `selfAnchor()` | `EntityAnchor?` | FEET or EYES anchor on the player, null when unmapped — [Packet enums](enums.md) |

## S2CMoveMinecartAlongTrackPacket

Minecart rail movement. The step list is not exposed.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | minecart entity network id |

## S2CPlayerAbilitiesPacket

Local player ability flags and speeds.

| Component | Type | Description |
|---|---|---|
| `invulnerable()` | `boolean` | damage immunity |
| `flying()` | `boolean` | currently flying |
| `allowFlying()` | `boolean` | flight permitted |
| `creativeMode()` | `boolean` | creative instant break |
| `flySpeed()` | `float` | flight speed factor, vanilla default 0.05 |
| `walkSpeed()` | `float` | walk speed factor, vanilla default 0.1 |

## S2CPlayerActionResponsePacket

Acknowledges a sequenced interaction so the client drops its prediction.

| Component | Type | Description |
|---|---|---|
| `sequence()` | `int` | acknowledged interaction sequence number |

## S2CPlayerPositionLookPacket

Server teleport of the local player, answered with `C2STeleportConfirmPacket`.

| Component | Type | Description |
|---|---|---|
| `teleportId()` | `int` | id to echo back in the confirm packet |
| `x()` | `double` | target x in blocks |
| `y()` | `double` | target y in blocks, feet level |
| `z()` | `double` | target z in blocks |
| `yaw()` | `float` | target yaw in degrees |
| `pitch()` | `float` | target pitch in degrees |

The relative-flag set is not exposed, so any of the six values may be a delta on the current one.

## S2CPlayerRespawnPacket

Local player respawn or dimension change.

| Component | Type | Description |
|---|---|---|
| `dimension()` | `String` | dimension id with namespace |
| `seed()` | `long` | hashed world seed for biome-dependent client effects |
| `gameMode()` | `GameMode?` | new game mode, null when unmapped — [Entities and filters](../../game/entities.md) |
| `lastGameMode()` | `GameMode?` | previous game mode, null when unmapped |
| `isDebug()` | `boolean` | debug world type |
| `isFlat()` | `boolean` | superflat world type |
| `portalCooldown()` | `int` | remaining portal cooldown in ticks |
| `seaLevel()` | `int` | dimension sea level in blocks |
| `flag()` | `byte` | bit 1 keeps attributes, bit 2 keeps tracked data |

## S2CPlayerRotationPacket

Server sets the local player rotation.

| Component | Type | Description |
|---|---|---|
| `yaw()` | `float` | yaw in degrees |
| `relativeYaw()` | `boolean` | yaw is a delta on the current yaw |
| `pitch()` | `float` | pitch in degrees |
| `relativePitch()` | `boolean` | pitch is a delta on the current pitch |

## S2CPlayerSpawnPositionPacket

World spawn or respawn anchor point.

| Component | Type | Description |
|---|---|---|
| `dimension()` | `String` | dimension id with namespace |
| `x()` | `int` | spawn block x |
| `y()` | `int` | spawn block y |
| `z()` | `int` | spawn block z |
| `yaw()` | `float` | spawn yaw in degrees |
| `pitch()` | `float` | spawn pitch in degrees |

## S2CProjectilePowerPacket

Acceleration power of a fireball-type projectile.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | projectile entity network id |
| `accelerationPower()` | `double` | acceleration factor applied per tick |

## S2CRemoveEntityStatusEffectPacket

Status effect cleared from an entity. Which effect is not exposed.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | affected entity network id |

## S2CSetCameraEntityPacket

Render camera switched to another entity.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | entity network id to view from |

## S2CVehicleMovePacket

Server moves the vehicle the player rides. The entity id is not exposed.

| Component | Type | Description |
|---|---|---|
| `x()` | `double` | vehicle x in blocks |
| `y()` | `double` | vehicle y in blocks |
| `z()` | `double` | vehicle z in blocks |
| `yaw()` | `float` | vehicle yaw in degrees |
| `pitch()` | `float` | vehicle pitch in degrees |
