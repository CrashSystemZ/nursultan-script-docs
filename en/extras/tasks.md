# Timers and tasks

`timer()` returns a fresh independent stopwatch on every call. Scheduled actions run on the client thread and only while the script is switched on.

```kotlin
val sinceAttack = timer()

on<ClientTickEvent> {
    if (sinceAttack.passedAndReset(10)) interaction.attackCrosshair()
}

everyTicks(100) { chat.print("still here") }
```

## Timers

| Method | Type | Description |
|---|---|---|
| `timer()` | `Timer` | a new independent stopwatch, already started |
| `elapsedTicks()` | `int` | client ticks since the last reset, clamped to 0..2147483647 |
| `elapsedMillis()` | `long` | wall-clock milliseconds since the last reset, never negative |
| `passed(ticks)` | `boolean` | true when `elapsedTicks() >= ticks` |
| `passedMillis(ms)` | `boolean` | true when `elapsedMillis() >= ms` |
| `passedAndReset(ticks)` | `boolean` | `passed(ticks)`, and resets when it returns true |
| `passedMillisAndReset(ms)` | `boolean` | `passedMillis(ms)`, and resets when it returns true |
| `reset()` | `void` | restarts both the tick and millisecond baselines |

Timer fields are volatile, so a timer can be read and reset off the client thread.

## Scheduling

| Method | Type | Description |
|---|---|---|
| `nextTick(action)` | `Task` | runs once after 1 client tick |
| `afterTicks(ticks, action)` | `Task` | runs once after that many client ticks (throws `ScriptException` when ticks < 0) |
| `everyTicks(ticks, action)` | `Task` | repeats every `ticks` client ticks, first run after 2×`ticks` (throws `ScriptException` when ticks < 1) |
| `client.tasks().nextTick(action)` | `Task` | `Tasks` form of `nextTick` |
| `client.tasks().afterTicks(ticks, action)` | `Task` | `Tasks` form of `afterTicks` |
| `client.tasks().everyTicks(ticks, action)` | `Task` | `Tasks` form of `everyTicks` |

`afterTicks(0, ...)` is accepted and the task is dropped before it ever runs; a null action throws `NullPointerException`.
A scheduled action is skipped while the script is off or unloaded, a throw is reported to the script console instead of propagating, and every outstanding task is cancelled on unload.

## The handle

| Method | Type | Description |
|---|---|---|
| `cancel()` | `void` | stops further runs and drops the handle from the script |
| `cancelled()` | `boolean` | true once cancelled |
| `close()` | `void` | calls `cancel()`, `Task` is `AutoCloseable` |

## Getting onto the client thread

| Method | Type | Description |
|---|---|---|
| `onClientThread(action)` | `Unit` | hands the action to the client thread, inline when already on it |
| `client.tasks().onClientThread(action)` | `void` | `Tasks` form of `onClientThread` |

`PacketReceiveEvent` (netty IO thread) and `PacketSendEvent` (the sending thread) are the only events that do not fire on the client thread — see [Packets](../actions/packets.md).
World, player, inventory, container, interaction and rotation calls throw `ScriptThreadException` off the client thread.

## Time

| Method | Type | Description |
|---|---|---|
| `client.tick()` | `long` | client ticks since launch, never resets in a session |
| `client.millis()` | `long` | wall-clock milliseconds, loses precision once stored in a `float` |
| `client.nanos()` | `long` | monotonic nanoseconds, `System.nanoTime()` |
| `client.fps()` | `int` | frames per second reported by the client |
| `client.tickDelta()` | `float` | render progress between the last two ticks, 0..1 |
| `client.onClientThread()` | `boolean` | true when the caller runs on the client thread |
