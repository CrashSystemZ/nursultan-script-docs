# Rotations

`rotations` is `client.rotations()`. An applied rotation is written into the next movement packet, so the server sees it; it is quantised to the mouse-sensitivity step.

```kotlin
on<PrePlayerTickEvent> {
    val target = combat.target() ?: return@on
    rotations.apply(
        rotations.lookAt(target),
        RotationOptions.DEFAULT.priority(RotationPriority.NOW)
    )
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

`Rotation` itself is on [Vectors, boxes, angles](../game/math.md#rotation); the attack points are on [Interaction](interaction.md#where-to-hit).

## Applying it

| Method | Type | Description |
|---|---|---|
| `rotations.apply(rotation)` | `void` | queues it with `RotationOptions.DEFAULT` (main thread only) |
| `rotations.apply(rotation, options)` | `void` | queues it, null options mean `DEFAULT` (main thread only) |

Applied at the next pre-player-tick and snapped to the mouse quantum with dither; one call covers one tick.
Does nothing while another rotation holds the handler lock, and throws `ScriptStateException` with no player or world.

## Options

| Method | Type | Description |
|---|---|---|
| `RotationOptions.DEFAULT` | `RotationOptions` | static field `(NORMAL, false, false, false, false, FAST)` |
| `options.priority()` | `RotationPriority` | ordering slot inside the tick's rotation queue |
| `options.priority(value)` | `RotationOptions` | copy with a new priority |
| `options.strongCorrection()` | `boolean` | true skips the WASD remap, movement follows the spoofed yaw |
| `options.strongCorrection(value)` | `RotationOptions` | copy with a new value |
| `options.smoothBackRotation()` | `boolean` | true returns the head gradually, false snaps it back |
| `options.smoothBackRotation(value)` | `RotationOptions` | copy with a new value |
| `options.backRotation()` | `BackRotation` | shape of the return, read only while `smoothBackRotation` is true |
| `options.backRotation(value)` | `RotationOptions` | copy with a new value |
| `options.clientSide()` | `boolean` | stored flag (deprecated, no effect: the rotation always reaches the server) |
| `options.clientSide(value)` | `RotationOptions` | copy with a new value (deprecated, no effect: the rotation always reaches the server) |
| `options.normalizeMouseMovement()` | `boolean` | stored flag (deprecated, no effect: the angle is always snapped to a mouse step) |
| `options.normalizeMouseMovement(value)` | `RotationOptions` | copy with a new value (deprecated, no effect: the angle is always snapped to a mouse step) |
| `RotationOptions(priority, clientSide, strongCorrection, smoothBackRotation, normalizeMouseMovement, backRotation)` | `RotationOptions` | canonical constructor (throws `NullPointerException` when `priority` or `backRotation` is null) |

The record is immutable: every setter returns a copy and `DEFAULT` never changes.
The four deprecated members are indexed under [Things that no longer do anything](../extras/api-versions.md#things-that-no-longer-do-anything).

## Order inside a tick

### RotationPriority

| Constant | Description |
|---|---|
| `NOW` | weight 200, applied last, overwrites lower priorities |
| `NORMAL` | weight 0, the value in `DEFAULT` |
| `LATER` | weight -200, applied first |

Rotations queued in one tick are applied `LATER` → `NORMAL` → `NOW`, so the highest priority is written last.

### BackRotation

| Constant | Description |
|---|---|
| `FAST` | 50% lerp per tick toward the camera, snaps after 20 ticks |
| `SMOOTH` | up to 12 degrees per tick per axis, snaps after 26 ticks |
