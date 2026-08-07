# Subscribing

An event is a moment in the game your script can react to: a tick happened, the player attacked, a packet arrived, a frame was drawn.

## How to subscribe

```kotlin
on<ClientTickEvent> {
    // every game tick
}

on<AttackEvent> { e ->
    chat.print("hitting " + e.target().name())
}
```

The event type goes in angle brackets and the object arrives in the lambda. Leave the parameter unnamed and it is available as `it`.

A subscription lives while the script is **on**. Switch it off and handlers unsubscribe; switch it back on and they return. There is nothing to clean up by hand.

## Priority

When several modules and scripts listen to the same event, priority decides the order:

```kotlin
on<MoveInputEvent>(priority = Priority.LAST) {
    it.sprint(false)
}
```

| Priority | When it runs |
|---|---|
| `FIRST` | before everyone |
| `EARLY` | |
| `NORMAL` | the default |
| `LATE` | |
| `LAST` | after everyone |

`LAST` is for having the final word — forcing input after everything else, say. `FIRST` is for cancelling an event before anyone else sees it.

## Cancelling

Some events can be cancelled, and then the game does not do what it was about to:

```kotlin
on<UseItemEvent> { e ->
    if (inventory.held().isA("ender_pearl")) {
        e.cancel()          // do not let it be used
    }
}
```

Cancellable events are marked in the [list](reference.md). By default a cancelled event never reaches your handler — if you want to see it anyway, say so:

```kotlin
on<AttackEvent>(ignoreCancelled = true) { e ->
    chat.print("someone cancelled the attack on " + e.target().name())
}
```

## Unsubscribing early

`on` returns a subscription you can drop by hand:

```kotlin
val sub = on<ClientTickEvent> { ... }

sub.unsubscribe()
sub.active()
```

You rarely need this — handlers already go away when the script is switched off.

## Which thread this runs on

Almost every event arrives on the client thread, where reading the world and acting on it is fine.

The exception is **packets**: `PacketReceiveEvent` and `PacketSendEvent` fire on the network thread. Reading the packet's fields there is fine; touching the world, the player or the inventory is not. Hop over first:

```kotlin
on<PacketReceiveEvent> { e ->
    val packet = e.packet()
    if (packet !is S2CEntityVelocityPacket) return@on
    onClientThread {
        control.jump()
    }
}
```

More in [Packets](../actions/packets.md).

## Do not freeze the game

A handler has 250 ms to finish. Miss that and the client decides the script hung, switches it off and prints the file and line to the console. The game keeps running.

So: no endless loops, and no "recompute everything" inside a tick. Split heavy work across ticks with [`everyTicks`](../extras/tasks.md).
