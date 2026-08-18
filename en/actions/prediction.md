# Prediction

`prediction` is `client.prediction()`. It simulates the local player forward on a shadow entity that copies the player's movement attributes, and it simulates projectile flight from the last sent position.

```kotlin
on<Render3DEvent> { e ->
    val fall = prediction.path(10) { it.onGround() }   // stops on the first grounded tick
    val landing = fall.last().position()
    e.render().box(Box.around(landing, 0.2), Colors.RED, true)
}
```

## Simulating movement

| Method | Type | Description |
|---|---|---|
| `prediction.after(ticks)` | `PredictionResult` | state after `ticks` simulated ticks, live keyboard input |
| `prediction.after(ticks, input)` | `PredictionResult` | same, with `input` held for every simulated tick |
| `prediction.path(ticks)` | `List<PredictionResult>` | one immutable entry per simulated tick, tick 0 excluded |
| `prediction.path(ticks, input)` | `List<PredictionResult>` | same, with a fixed input |
| `prediction.path(ticks, until)` | `List<PredictionResult>` | stops after the first entry matching `until`, that entry included |
| `prediction.path(ticks, input, until)` | `List<PredictionResult>` | fixed input plus stop predicate |

`ticks` is 1..100 on every method; movement runs on a shadow entity copying movement speed, sneak speed, jump strength, gravity, step height and movement efficiency.
All six need a world and the client thread (main thread only); an out-of-range `ticks` throws `ScriptException`, a call from inside a `path` predicate throws `ScriptStateException`.

## One simulated tick

| Method | Type | Description |
|---|---|---|
| `result.position()` | [`Vec`](../game/math.md#vec) | predicted feet position in world blocks |
| `result.velocity()` | [`Vec`](../game/math.md#vec) | predicted velocity in blocks per tick |
| `result.box()` | [`Box`](../game/math.md#box) | predicted bounding box in world coordinates |
| `result.onGround()` | `boolean` | predicted on-ground flag |
| `result.inWater()` | `boolean` | predicted touching-water flag |
| `result.sneaking()` | `boolean` | sneak state used for this tick |
| `result.sprinting()` | `boolean` | sprint state used for this tick |
| `result.jumping()` | `boolean` | jump input used for this tick |
| `result.fallDistanceBlocks()` | `double` | accumulated fall distance in blocks |

## The input

| Method | Type | Description |
|---|---|---|
| `moveInput(forward, backward, left, right, jump, sneak, sprint)` | `MoveInput` | builds the input record, every argument defaults to `false` |
| `MoveInput.NONE` | `MoveInput` | all seven flags false |
| `input.forward()` | `boolean` | forward key held |
| `input.backward()` | `boolean` | backward key held |
| `input.left()` | `boolean` | strafe-left key held |
| `input.right()` | `boolean` | strafe-right key held |
| `input.jump()` | `boolean` | jump key held |
| `input.sneak()` | `boolean` | sneak key held |
| `input.sprint()` | `boolean` | sprint key held |
| `input.copy(forward, backward, left, right, jump, sneak, sprint)` | `MoveInput` | copy with named overrides, defaults read from the receiver |

`MoveInputEvent.toInput()` returns the keys currently held — see [Event list](../events/reference.md).

## Projectiles

| Method | Type | Description |
|---|---|---|
| `prediction.projectile(kind)` | `ProjectilePath` | trajectory on the last sent rotation, kind's default power |
| `prediction.projectile(kind, aim)` | `ProjectilePath` | `aim` null means the last sent rotation, kind's default power |
| `prediction.projectile(kind, aim, power)` | `ProjectilePath` | explicit launch power |
| `path.points()` | `List<Vec>` | trajectory positions, one per tick, from the launch point |
| `path.lands()` | `boolean` | a block or entity hit was found |
| `path.landingPosition()` | `Vec?` | position of the hit, null when nothing was hit |
| `path.hitEntity()` | [`Entity?`](../game/entities.md) | entity struck at the landing point, else null |
| `path.hitsBlock()` | `boolean` | the landing hit is a block |
| `path.flightTicks()` | `int` | `points().size() - 1`, minimum 0 |

Origin is the last sent position plus eye height, the launch velocity includes the player's current movement, and `aim` is a [`Rotation`](../game/math.md#rotation).
All three need a world and the client thread (main thread only); a null `kind` or `power` at or below 0 throws `ScriptException`.

### ProjectileKind

| Constant | Description |
|---|---|
| `ARROW` | arrow physics, default power 3.0 |
| `TRIDENT` | trident physics, default power 2.5 |
| `ENDER_PEARL` | ender pearl physics, default power 1.5 |
| `SPLASH_POTION` | splash potion physics, default power 0.5, aim pitch -20° |
| `SNOWBALL` | snowball physics, default power 1.5 |
| `WIND_CHARGE` | wind charge physics, default power 1.5 |
