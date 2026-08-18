# Movement

`control` is `game.control()` — three direct writes onto the local player. `gameSettings` is `game.settings()` and reads or writes a slice of the vanilla options.

```kotlin
on<ClientTickEvent> {
    control.sprinting(true)
}

on<MoveInputEvent> {
    it.forward(true)   // hold forward for this tick
    it.jump(false)
}
```

## Direct commands

| Method | Type | Description |
|---|---|---|
| `control.sprinting(value)` | `void` | sets the local player's sprint flag (main thread only) |
| `control.sneaking(value)` | `void` | sets the local player's sneak flag (main thread only) |
| `control.jump()` | `void` | jumps, no-op while not on the ground (main thread only) |

Every method throws `ScriptThreadException` off the client thread and `ScriptStateException` with no world loaded.

## Rewriting input

The tick's seven raw movement flags are [`MoveInputEvent`](../events/reference.md#moveinputevent); the game rebuilds the player's input from them after the handler returns. The frame's mouse deltas are [`LookInputEvent`](../events/reference.md#lookinputevent).

What is physically held right now, independent of the tick's input, is [Keys and binds](keys.md); aiming the head at a point is [Rotations](rotations.md).

## Game settings

| Method | Type | Description |
|---|---|---|
| `gameSettings.perspective()` | `Perspective` | the player's chosen point of view, `FIRST_PERSON` when options are unavailable (API 2) |
| `gameSettings.perspective(value)` | `void` | sets the point of view, null is a no-op (API 2) (main thread only) |
| `gameSettings.scaleFactor()` | `double` | window GUI scale factor, e.g. 1.0 / 2.0 / 3.0 (API 2) |
| `gameSettings.mouseSensitivity()` | `double` | sensitivity option 0..1, 0.5 when options are unavailable (API 2) |
| `gameSettings.mouseSensitivity(value)` | `void` | sets sensitivity, clamped to 0..1 (API 2) (main thread only) |

Neither setting is restored when the script switches off; `scaleFactor()` is the number [2D render](../ui/render-2d.md) pixels are divided by to get vanilla interface coordinates.
`PerspectiveEvent` fires on every point-of-view change, whoever caused it — [Event list](../events/reference.md#perspectiveevent).

**`Perspective`**

| Method | Type | Description |
|---|---|---|
| `perspective.firstPerson()` | `boolean` | true only for `FIRST_PERSON` (API 2) |
| `perspective.thirdPerson()` | `boolean` | negation of `firstPerson()` (API 2) |
| `perspective.frontView()` | `boolean` | true only for `THIRD_PERSON_FRONT` (API 2) |

| Constant | Description |
|---|---|
| `FIRST_PERSON` | camera in the player's head |
| `THIRD_PERSON_BACK` | camera behind the player |
| `THIRD_PERSON_FRONT` | camera in front of the player, facing it |

## Slowdown

The movement multiplier the game applies while an item is in use is [`SlowdownEvent`](../events/reference.md#slowdownevent); the value written back replaces the vanilla one and is not clamped.
