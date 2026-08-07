# Your first script

## An empty file already works

Create `hello.kts` and write one line:

```kotlin
chat.print("hi")
```

Drop the file into the scripts folder, open the menu → **Scripts**, flip the switch — "hi" lands in chat.

Everything you write at the top level of the file runs **once, on load**. That is usually where you declare settings and subscribe to events; the actual work happens inside handlers.

## Add settings and a bind

```kotlin
name("AutoJump")
description("Jumps for you")

key(Key.G)

val delay = slider("Delay", 5, 1, 20)
val sound = checkBox("Sound", true)

var ticks = 0

onEnable { ticks = 0 }

on<ClientTickEvent> {
    if (!inGame) return@on
    if (++ticks < delay.intValue()) return@on
    ticks = 0
    control.jump()
    if (sound.value()) {
        world.playSound("minecraft:entity.experience_orb.pickup", 1f, 1f)
    }
}
```

What is going on here:

* `name` and `description` are how the script is labelled in the menu. Skip them and the file name is used.
* `key(Key.G)` is the default bind. If the player already rebound it, their choice is not overwritten.
* `slider` and `checkBox` are settings — they show up in the script's card.
* `onEnable` runs when the script is switched on, handy for resetting counters.
* `on<ClientTickEvent>` is a handler; it works while the script is on.

## Two ways to read a setting

If you only need the value, take it through `by`:

```kotlin
val delay by slider("Delay", 5f, 1f, 20f)   // delay is a Float

if (ticks >= delay) { ... }
```

If you need the object itself — to listen for changes or hide it conditionally — keep it as is:

```kotlin
val delay = slider("Delay", 5f, 1f, 20f)

delay.value()                       // read
delay.value(10f)                    // write
delay.onChange { chat.print("now $it") }
```

More in [Kinds of settings](../settings/types.md).

## Where to look when something breaks

Everything a script prints, and every error it throws, goes to the script console together with the file and line where it happened:

```
[15:03:31] [ERROR] [auto-jump] failed - auto-jump.kts:12: NullPointerException
```

Open it from the menu → **Scripts** → the terminal button next to the search field, and pin it if you want it to stay while you tab back into the game. More in [Messages](../ui/messages.md).

Five throws in a row and the script switches itself off so it stops spamming. More in [Sandbox and limits](../extras/limits.md).
