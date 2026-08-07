# Rotations

`rotations` turns the player's head. You normally want it so a hit or a block placement goes where it should, not where the player happens to look.

## Reading the current angles

```kotlin
rotations.player()      // where the player looks
rotations.camera()      // where the camera looks
rotations.lastSent()    // what was last sent to the server
```

The difference between `player()` and `lastSent()` matters: a script can send the server one angle while the head on screen stays put.

## Where to look

```kotlin
val aim = rotations.lookAt(target.position())
val aim = rotations.lookAt(target)            // straight at an entity
```

A `Rotation` is a pair of angles, `yawDegrees` and `pitchDegrees`:

```kotlin
aim.yawDegrees()
aim.pitchDegrees()
aim.direction()                     // Vec it points at
aim.angleTo(other)                  // how many degrees you still have to turn
aim.yawDeltaTo(other)
aim.pitchDeltaTo(other)
```

`angleTo` is handy for not hitting while the target is still far off the crosshair:

```kotlin
if (rotations.player().angleTo(aim) > 30f) return@on
```

## Applying it

```kotlin
rotations.apply(aim)
```

One call covers the current tick. To hold a rotation, call it every tick:

```kotlin
on<PrePlayerTickEvent> {
    val target = combat.target() ?: return@on
    rotations.apply(rotations.lookAt(combat.attackPoint(target)))
}
```

`rotations.angleTo(point)` is the short way to ask how far off a target is without building a `Rotation` yourself.

## Rotation options

When you need control over how the head turns:

```kotlin
rotations.apply(
    aim,
    RotationOptions.DEFAULT
        .priority(RotationPriority.NOW)
        .clientSide(false)
        .backRotation(BackRotation.SMOOTH)
)
```

| Option | What it does |
|---|---|
| `priority` | who wins when several modules ask: `NOW`, `NORMAL`, `LATER` |
| `clientSide` | turn on screen only, send nothing to the server |
| `strongCorrection` | pull movement harder towards the new angle |
| `smoothBackRotation` | ease the head back instead of snapping |
| `normalizeMouseMovement` | round the angle to a mouse step |
| `backRotation` | how to come back: `FAST` or `SMOOTH` |

Every method returns a new object, so you can chain them, and `RotationOptions.DEFAULT` stays untouched.

## Who wins

Built-in modules and scripts both ask for rotations. The higher priority wins; on a tie, whoever asked later in the tick does. So reach for `NOW` only when you really need to beat everyone else.
