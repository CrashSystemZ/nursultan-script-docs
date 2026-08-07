# Timers and tasks

## Timers

Counting ticks by hand is the thing scripts get wrong most often. That is what a timer is for:

```kotlin
val sinceAttack = timer()

on<ClientTickEvent> {
    if (!sinceAttack.passed(10)) return@on
    // ten ticks since the last reset
    sinceAttack.reset()
}
```

| Method | What it does |
|---|---|
| `passed(ticks)` | have that many ticks passed |
| `passedMillis(ms)` | the same in milliseconds |
| `passedAndReset(ticks)` | check and reset in one go |
| `passedMillisAndReset(ms)` | the same in milliseconds |
| `elapsedTicks()` | how many ticks passed |
| `elapsedMillis()` | how many milliseconds passed |
| `reset()` | start counting again |

`passedAndReset` saves a line when you were going to reset anyway:

```kotlin
if (sinceAttack.passedAndReset(10)) {
    interaction.attackCrosshair()
}
```

You can have as many timers as you like — one per action with its own delay.

## Delaying and repeating

```kotlin
nextTick { ... }                  // next tick
afterTicks(20) { ... }            // a second from now
val task = everyTicks(100) { ... } // every five seconds
```

All of these run on the client thread.

`everyTicks` returns a task you can stop:

```kotlin
task.cancel()
task.cancelled()
```

You do not clean tasks up yourself — they go away when the script unloads.

A task only fires while the script is switched on, the same as handlers and commands: a loaded but disabled script does nothing at all. The schedule itself keeps running, so after you switch the script back on the task carries on without being re-created.

That also means you cannot defer cleanup out of `onDisable` — by the time it runs the script counts as off, and an `afterTicks` block scheduled there would never fire. Clean up right there in `onDisable`.

## Timer or task

| You want | Use |
|---|---|
| "no more than once every N ticks" inside a handler | a timer |
| "do this once, N ticks from now" | `afterTicks` |
| "do this regularly, independent of other handlers" | `everyTicks` |
| "finish this next tick" | `nextTick` |

## Getting onto the client thread

You cannot touch the world from a packet handler — that is another thread. Hop over like this:

```kotlin
on<PacketReceiveEvent> { e ->
    onClientThread {
        control.jump()
    }
}
```

If you are already on the client thread the block runs straight away. `client.onClientThread()` tells you where you are.

## Time

```kotlin
client.tick()             // ticks the client has lived
client.millis()           // system time
client.nanos()
client.fps()
client.tickDelta()        // the fraction of a frame between ticks
```

For delays prefer `client.tick()` and timers over `millis()`: the game runs on ticks.
