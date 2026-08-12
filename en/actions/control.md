# Movement

There are two ways to move the player: a direct command right now, or rewriting the input for this tick.

## Direct commands

```kotlin
control.jump()
control.sprinting(true)
control.sneaking(true)
```

`jump()` jumps immediately if the player is on the ground. `sprinting` and `sneaking` turn the state on and off.

Good for one-off actions: hop on knockback, crouch for a moment.

## Rewriting input

`MoveInputEvent` is what the game is about to do this tick. Every field can be rewritten:

```kotlin
on<MoveInputEvent> { e ->
    e.forward(true)
    e.sprint(true)
    e.jump(false)
}
```

| Field | What it means |
|---|---|
| `forward()`, `backward()` | forward and back |
| `left()`, `right()` | strafing |
| `jump()` | jump |
| `sneak()` | sneak |
| `sprint()` | sprint |

Good for holding a state: keeping sprint off, walking the player forward, blocking a jump.

## Priority

Client modules and scripts both want the same input. To have the final word, subscribe with `Priority.LAST`:

```kotlin
on<MoveInputEvent>(priority = Priority.LAST) {
    it.sprint(false)
}
```

For example, killing sprint for exactly one tick so the hit lands as a critical:

```kotlin
var blockSprintTicks = 0

on<MoveInputEvent>(priority = Priority.LAST) {
    if (blockSprintTicks > 0) {
        blockSprintTicks--
        it.sprint(false)
    }
}

// somewhere in the combat logic
blockSprintTicks = 1
```

## The mouse

`LookInputEvent` gives you mouse movement for the frame, and it can be rewritten too:

```kotlin
on<LookInputEvent> { e ->
    e.deltaX(e.deltaX() * 0.5)     // half speed horizontally
    e.deltaY(e.deltaY() * 0.5)
}
```

Cancel the event and the mouse stops turning the camera at all.

Turning the head towards a target is not this — that is [Rotations](rotations.md).

## Game settings

`gameSettings` is Minecraft's own options — what the player set in the game menu:

```kotlin
gameSettings.perspective()                               // FIRST_PERSON | THIRD_PERSON_BACK | THIRD_PERSON_FRONT
gameSettings.perspective(Perspective.THIRD_PERSON_BACK)  // exactly what pressing F5 does

gameSettings.scaleFactor()                               // GUI Scale as a number: 1.0, 2.0, 3.0 …
gameSettings.mouseSensitivity()                          // 0..1, where 0.5 is the "100%" the game shows
gameSettings.mouseSensitivity(0.5)
```

A `Perspective` answers three questions about itself: `firstPerson()`, `thirdPerson()`, `frontView()`.

Writing any of these has to happen on the client thread — inside a handler you already are there, from anywhere else wrap it in `onClientThread { }`. Nothing is restored when your script switches off, so put it back yourself:

```kotlin
var previous = gameSettings.perspective()

onEnable {
    previous = gameSettings.perspective()
    gameSettings.perspective(Perspective.THIRD_PERSON_BACK)
}

onDisable { gameSettings.perspective(previous) }
```

`PerspectiveEvent` fires whenever the view changes, no matter who changed it — the player, your script, or another one:

```kotlin
on<PerspectiveEvent> { e ->
    log.info("${e.previous()} -> ${e.current()}")
}
```

What you read is always the player's own choice. Client modules that force a view — FreeCam, FreeLook — change how the frame is drawn, not the option, so `gameSettings.perspective()` keeps telling you what the player picked.

`scaleFactor()` is the bridge between [2D render](../ui/render-2d.md), which works in real framebuffer pixels, and the vanilla interface, which is those pixels divided by this number.

## Asking a key directly

Events tell you when a key *changed*. `keys` tells you what is held **right now**, which is what a HUD needs:

```kotlin
on<Render2DEvent> { e ->
    if (keys.isDown(Key.LEFT_SHIFT)) {
        e.render().text("sneaking", 10f, 10f, 10f, Colors.WHITE)
    }
}
```

```kotlin
keys.isDown(Key.G)
keys.mouseDown(0)          // 0 left, 1 right, 2 middle
keys.mouseX()  keys.mouseY()   // cursor, in the same pixels Render2DEvent draws in
keys.cursorLocked()        // false while a screen is open
keys.lockCursor()
keys.unlockCursor()
keys.inputBlocked()        // the player is typing in chat or a text field
```

Check `inputBlocked()` before reacting to letter keys, or your script fires while the player writes a message.

This reads the hardware, so it is `true` even while a screen is open and the game itself ignores the key. `MoveInputEvent` is the opposite: it is what the game decided to do this tick, already filtered.

## Slowdown

`SlowdownEvent` fires when the game is about to slow movement down: food, a shield, a drawn bow. The multiplier can be rewritten:

```kotlin
on<SlowdownEvent> { e ->
    e.multiplier(1f)     // do not slow down at all
}
```
