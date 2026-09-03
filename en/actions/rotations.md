# Rotations

`rotations` is `client.rotations()`. An applied rotation is written verbatim into the next movement packet, so the server sees it; the client no longer snaps it to the mouse grid for you, and `rotations.quantized(rotation)` is what gives you the angle that will actually be sent.

```kotlin
on<PrePlayerTickEvent> {
    val target = combat.target() ?: return@on
    val aim = rotations.quantized(rotations.lookAt(target))
    rotations.apply(aim, RotationOptions.DEFAULT.priority(RotationPriority.NOW))
}
```

## Reading angles

| Method | Type | Description |
|---|---|---|
| `rotations.player()` | `Rotation` | player yaw/pitch in degrees, spoof included |
| `rotations.camera()` | `Rotation` | angle the rotation handler holds, equals `player()` while idle |
| `rotations.lastSent()` | `Rotation` | yaw/pitch carried by the last movement packet |
| `rotations.angleTo(point)` | `float` | degrees 0..180 from the current look vector to a `Vec` |

Every method throws `ScriptStateException` with no player or world.

## Building one

| Method | Type | Description |
|---|---|---|
| `rotations.lookAt(point)` | `Rotation` | yaw/pitch from the eye position toward a `Vec` |
| `rotations.lookAt(entity)` | `Rotation` | yaw/pitch toward that entity's `NEAREST` attack point |
| `rotations.quantized(rotation)` | `Rotation` | copy snapped to the mouse-sensitivity grid (API 6) |

`quantized` moves both yaw and pitch onto the mouse step, unwraps the yaw into the player's frame and clamps the pitch to -90..90 degrees.
Raycasting the quantised value checks the exact angle you are about to apply.

`Rotation` itself is on [Vectors, boxes, angles](../game/math.md#rotation); the attack points are on [Interaction](interaction.md#where-to-hit).

## Applying it

| Method | Type | Description |
|---|---|---|
| `rotations.apply(rotation)` | `void` | queues it with `RotationOptions.DEFAULT` (main thread only) |
| `rotations.apply(rotation, options)` | `void` | queues it, null options mean `DEFAULT` (main thread only) |
| `rotations.locked()` | `boolean` | another rotation holds the handler, so `apply` does nothing right now |

Applied at the next pre-player-tick exactly as given; one call covers one tick.
Does nothing while another rotation holds the handler lock, and throws `ScriptStateException` with no player or world.

## Options

| Method | Type | Description |
|---|---|---|
| `RotationOptions.DEFAULT` | `RotationOptions` | static field `(NORMAL, false, BackRotations.SNAP)` |
| `options.priority()` | `RotationPriority` | ordering slot inside the tick's rotation queue |
| `options.priority(value)` | `RotationOptions` | copy with a new priority |
| `options.strongCorrection()` | `boolean` | true skips the WASD remap, movement follows the spoofed yaw |
| `options.strongCorrection(value)` | `RotationOptions` | copy with a new value |
| `options.backRotation()` | `BackRotation` | shape of the return to the camera |
| `options.backRotation(value)` | `RotationOptions` | copy with a new value |
| `options.smoothBackRotation()` | `boolean` | stored flag (deprecated, use `backRotation`) |
| `options.smoothBackRotation(value)` | `RotationOptions` | copy with a new value, true turns a `SNAP` return into `HUMANIZED` (deprecated, use `backRotation`) |
| `options.clientSide()` | `boolean` | stored flag (deprecated) (no effect: the rotation always reaches the server) |
| `options.clientSide(value)` | `RotationOptions` | copy with a new value (deprecated) (no effect: the rotation always reaches the server) |
| `options.normalizeMouseMovement()` | `boolean` | stored flag (deprecated, use `rotations.quantized`) (no effect: the value is never read) |
| `options.normalizeMouseMovement(value)` | `RotationOptions` | copy with a new value (deprecated, use `rotations.quantized`) (no effect: the value is never read) |
| `RotationOptions(priority, clientSide, strongCorrection, smoothBackRotation, normalizeMouseMovement, backRotation)` | `RotationOptions` | canonical constructor (throws `NullPointerException` when `priority` or `backRotation` is null) |
| `RotationOptions(priority, strongCorrection, backRotation)` | `RotationOptions` | constructor without the deprecated flags (API 6) |

The record is immutable: every setter returns a copy and `DEFAULT` never changes.
The deprecated members are indexed under [Things that no longer do anything](../extras/api-versions.md#things-that-no-longer-do-anything).

## Order inside a tick

### RotationPriority

| Constant | Description |
|---|---|
| `NOW` | weight 200, applied last, overwrites lower priorities |
| `NORMAL` | weight 0, the value in `DEFAULT` |
| `LATER` | weight -200, applied first |

Rotations queued in one tick are applied `LATER` → `NORMAL` → `NOW`, so the highest priority is written last.

## Returning the head

| Method | Type | Description |
|---|---|---|
| `BackRotations.SNAP` | `BackRotation` | 1 tick, returns the head at once, the value in `DEFAULT` (API 6) |
| `BackRotations.INSTANT` | `BackRotation` | 2 ticks, half the delta then the rest (API 6) |
| `BackRotations.HUMANIZED` | `BackRotation` | 20 ticks of recorded human deltas (API 6) |
| `BackRotation.FAST` | `BackRotation` | the return `BackRotations.SNAP` gives (deprecated, use `BackRotations.SNAP`) |
| `BackRotation.SMOOTH` | `BackRotation` | the return `BackRotations.HUMANIZED` gives (deprecated, use `BackRotations.HUMANIZED`) |

Once nothing applies a rotation any more, the handler walks the spoofed head back to the camera with the `BackRotation` of the last applied options.

### Writing your own

| Method | Type | Description |
|---|---|---|
| `BackRotation.step(from, to, tick)` | `Rotation?` | angle for this tick, null when the return is finished (API 6) |
| `BackRotation.maxTicks()` | `int` | ticks the return may take, 20 by default (API 6) |
| `backRotation(maxTicks) { from, to, tick -> }` | `BackRotation` | DSL form of the interface, `maxTicks` defaults to 20 (API 6) |

`from` is where the spoofed head points, `to` is where the real camera points, `tick` counts from 0.
Your steps are applied verbatim, so quantise them yourself; the handler ends the return once `maxTicks` is reached.
