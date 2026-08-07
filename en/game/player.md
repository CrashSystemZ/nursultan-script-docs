# Your player

`player` is you. Everything any [entity](entities.md) has, it has too, plus what only you can see about yourself.

Data is read at the moment you ask: `player.health()` gives you health right now, so do not cache values between ticks.

Almost none of it means anything outside a world, so start with a check:

```kotlin
on<ClientTickEvent> {
    if (!inGame) return@on
    chat.print("health " + player.health())
}
```

or shorter:

```kotlin
on<ClientTickEvent> {
    whenInGame {
        chat.print("health " + player.health())
    }
}
```

## Where I am and where I look

```kotlin
player.position()            // Vec of the feet
player.eyePosition()         // Vec of the eyes
player.x()  player.y()  player.z()
player.previousPosition()    // where it was last tick
player.renderPosition()      // where it is drawn this frame

player.rotation()            // Rotation
player.yaw()  player.pitch()
player.velocity()            // Vec of the velocity
```

`renderPosition()` is what you want for drawing: it is already smoothed between ticks. `position()` is the "real" one for logic.

## State

```kotlin
player.health()              // usually 0..20
player.maxHealth()
player.absorption()          // golden hearts
player.armorPoints()
player.food()
player.saturation()

player.alive()
player.onGround()
player.sneaking()
player.sprinting()
player.wasSprinting()        // was it sprinting last tick
player.inWater()
player.onFire()
player.flying()              // creative flight
player.gliding()             // elytra
player.climbing()            // on a ladder
player.riding()              // on a horse or a boat
player.creative()
player.gameMode()
player.pingMs()
player.fallDistanceBlocks()
player.hasMovementInput()    // are movement keys held
```

## Combat

```kotlin
player.attackCooldown()             // 0..1, where 1 is a full swing
player.belowMinimumAttackCharge()   // hitting now is pointless
player.entityReachBlocks()          // how far entities are reachable
player.blockReachBlocks()           // how far blocks are reachable
player.usingItem()                  // holding right click
player.usingHand()                  // with which hand
player.itemUseTicks()               // for how many ticks already
player.blocking()                   // blocking with a shield
player.hurtTicks()                  // how long ago it was hurt
```

The usual "can I hit" check:

```kotlin
if (player.attackCooldown() >= 0.9f && !player.belowMinimumAttackCharge()) {
    interaction.attack(target)
}
```

## Effects

```kotlin
player.hasEffect("speed")
player.effect("speed")?.amplifier()
player.effects()             // all of them
```

Each effect carries `id()`, `name()`, `amplifier()`, `durationTicks()`, `infinite()`, `beneficial()`.

## Items on you

```kotlin
player.mainHandItem()
player.offHandItem()
player.armorItem(ArmorSlot.HELMET)
player.armorItems()
player.isNaked()             // no armor at all
```

The whole inventory is in [Inventory](inventory.md).

## Odds and ends

```kotlin
player.respawn()             // press "Respawn"
player.xpLevel()
player.xpProgress()
player.airTicks()
```

## Vectors

Positions are `Vec` with ordinary arithmetic:

```kotlin
val target = player.position().add(0.0, 1.0, 0.0)
val distance = player.position().distanceTo(target)
val direction = target.subtract(player.position()).normalize()
```

`squaredDistanceTo` is cheaper than `distanceTo` — if you only need to compare distances, use it.
