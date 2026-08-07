# Prediction

Prediction answers "what happens if nothing changes": where I will be in a few ticks, and where a projectile will land.

## Where I will be

```kotlin
val soon = prediction.after(10)      // ten ticks from now

soon.position()
soon.velocity()
soon.box()
soon.onGround()
soon.inWater()
soon.sneaking()
soon.sprinting()
soon.jumping()
soon.fallDistanceBlocks()
```

It is worked out from your current velocity, input and world physics. Future player input is unknowable, so the further out you go the less it means. A dozen or two ticks is the practical ceiling.

For example, not jumping when you will end up in water five ticks from now anyway:

```kotlin
if (prediction.after(5).inWater()) return@on
```

## What if I pressed something else

The version above keeps your current keys held for the whole run. Pass a `MoveInput` and it runs the keys you name instead — that is how you ask "what would happen if I jumped from here" without jumping:

```kotlin
prediction.after(3, moveInput(forward = true, sprint = true, jump = true))
```

`moveInput(...)` takes `forward`, `backward`, `left`, `right`, `jump`, `sneak`, `sprint`, all `false` unless you say otherwise. To start from the keys that are actually down rather than from nothing, take them off the event and change the one you care about:

```kotlin
on<MoveInputEvent> { e ->
    val jumped = e.toInput().copy(jump = true)
    if (prediction.after(2, jumped).onGround()) {
        e.jump(true)
    }
}
```

`copy` works the same way settings do — everything you leave out stays as it was.

Everything else about you is still taken from the present: position, velocity, effects, attributes, the item you are using. Only the keys change.

## Every tick, not just the last one

`after(n)` hands you tick `n` and says nothing about the ticks on the way there. When the question is "at some point during", ask for the whole run:

```kotlin
prediction.path(10)                 // ten results, tick 1 through tick 10
prediction.path(10, someInput)      // the same, on keys you name
```

One simulation, one result per tick, in order. It is the movement twin of `projectile().points()`.

A jump up a staircase is the whole idea in one line. On flat ground you are still airborne two ticks after jumping; on a step you are already standing on it, so "will I be on the ground again almost immediately" is exactly the question that separates the two:

```kotlin
val landsFast = prediction.path(2, e.toInput().copy(jump = true)).any { it.onGround() }
```

Do not rebuild this out of `after` in a loop — `path(n)` runs the simulation once, `(1..n).map { after(it) }` runs it `n` times over.

## Stopping early

Asking for a hundred ticks does not mean paying for a hundred. Add a condition and the run stops on the first tick that satisfies it — that tick included, nothing after it:

```kotlin
val fall = prediction.path(100) { it.onGround() }

fall.size                    // how many ticks until you land
fall.last().position()       // where you land
```

If the condition never comes true you get the full `ticks` results, so `fall.last().onGround()` is what tells you whether it actually landed or just ran out of room.

The condition sees each tick as it happens, which is the point: `path(100) { it.inWater() }`, `path(60) { it.fallDistanceBlocks() > 3.0 }`, `path(40, someInput) { !it.sprinting() }`.

One rule about it: **do not call prediction from inside the condition.** The simulation runs on a single shared entity, so a nested call would quietly corrupt the run it is nested in. Doing it raises a `ScriptStateException` rather than returning wrong numbers.

## Where a projectile will go

```kotlin
val path = prediction.projectile(ProjectileKind.ARROW)

path.points()             // the trajectory as points
path.lands()              // will it get anywhere at all
path.landingPosition()    // where it lands
path.hitEntity()          // who it hits, or null
path.hitsBlock()
path.flightTicks()        // how many ticks it flies
```

Projectile kinds: `ARROW`, `TRIDENT`, `ENDER_PEARL`, `SPLASH_POTION`, `SNOWBALL`, `WIND_CHARGE`.

By default it uses your current angle and a full charge. You can give it your own:

```kotlin
prediction.projectile(ProjectileKind.ARROW, aim)
prediction.projectile(ProjectileKind.ARROW, aim, 0.7f)
```

## Only shoot when it lands

The most common use is not wasting a projectile:

```kotlin
fun willHitEnemy(kind: ProjectileKind): Boolean {
    val victim = prediction.projectile(kind).hitEntity() ?: return false
    return victim.alive() && !victim.isSelf() && !victim.isAlly()
}

on<ClientTickEvent> {
    if (!player.usingItem()) return@on
    if (!inventory.held().isA("trident")) return@on
    if (player.itemUseTicks() < 10) return@on
    if (!willHitEnemy(ProjectileKind.TRIDENT)) return@on
    interaction.stopUsingItem()
}
```

## Drawing the trajectory

`points()` is a ready-made list you can hand to the [3D render](../ui/render-3d.md):

```kotlin
on<Render3DEvent> { e ->
    val path = prediction.projectile(ProjectileKind.ENDER_PEARL)
    val points = path.points()
    for (i in 0 until points.size - 1) {
        e.render().line(points[i], points[i + 1], Colors.CYAN, true)
    }
    path.landingPosition()?.let {
        e.render().box(Box.around(it, 0.2), Colors.RED, true)
    }
}
```

## About the cost

Every call is a simulation, not a field read. Once or twice a tick is fine; inside a loop over every entity is not. If you need the result several times in a tick, compute it once and keep it in a variable.
