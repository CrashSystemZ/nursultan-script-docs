# Subscribing

`on<E> { }` registers a handler that lives as long as the script is switched on. Every script handler runs at one fixed priority: after the client's `BEFORE_ALL` and `BEFORE` modules, before its `NOW` ones.

```kotlin
on<ClientTickEvent> { }

on<UseItemEvent> { e ->
    if (inventory.held().isA("ender_pearl")) {
        e.cancel()
    }
}

val sub = on<AttackEvent>(ignoreCancelled = true) { e -> chat.print(e.target().name()) }
sub.unsubscribe()
```

## Registering a handler

| Method | Type | Description |
|---|---|---|
| `on<E>(ignoreCancelled) { }` | `Subscription` | subscribes by reified event type, `ignoreCancelled` defaults to false (throws `ScriptException` when `E` is not a supported event) |
| `on(type, ignoreCancelled) { }` | `Subscription` | subscribes by event class (throws `ScriptException` when the class is not a supported event) |
| `on<E>(priority, ignoreCancelled) { }` | `Subscription` | (deprecated, drop the argument) |
| `on(type, priority, ignoreCancelled) { }` | `Subscription` | (deprecated, drop the argument) |

`ignoreCancelled = true` skips the handler when the event is already cancelled.

### Events

| Method | Type | Description |
|---|---|---|
| `Events.on(type, handler)` | `Subscription` | subscribes with `EventOptions.DEFAULT` |
| `Events.on(type, options, handler)` | `Subscription` | subscribes with options; null options fall back to `DEFAULT` |
| `Events.supportedEvents()` | `List<String>` | simple names of every supported event, sorted |

`Events` is the interface the `on(...)` overloads above delegate to. A handler registered while the script is off is activated when it is switched on.

## Cancelling

| Method | Type | Description |
|---|---|---|
| `Cancellable.cancel()` | `void` | marks this dispatch cancelled |
| `Cancellable.cancelled()` | `boolean` | true after `cancel()` on this dispatch |
| `CancellableEvent.cancel()` | `void` | sets the flag, never resets it |
| `CancellableEvent.cancelled()` | `boolean` | flag value, false at dispatch start |

`CancellableEvent` is the base class of every cancellable event; the flag is cleared before every dispatch, so cancellation never leaks into the next one.
Which events are cancellable, and what cancelling each one skips: [Event list](reference.md).

## Options

| Method | Type | Description |
|---|---|---|
| `EventOptions(ignoreCancelled)` | `EventOptions` | builds options with `Priority.NORMAL` (API 2) |
| `EventOptions(priority, ignoreCancelled)` | `EventOptions` | canonical constructor (throws `NullPointerException` when priority is null) |
| `EventOptions.DEFAULT` | `EventOptions` | `Priority.NORMAL`, `ignoreCancelled = false` |
| `EventOptions.priority()` | `Priority` | (deprecated) (no effect: the value is never read) |
| `EventOptions.ignoreCancelled()` | `boolean` | true skips the handler on an already-cancelled event |
| `EventOptions.ignoreCancelled(value)` | `EventOptions` | copy with the flag replaced |
| `EventOptions.priority(priority)` | `EventOptions` | copy of `DEFAULT` carrying that priority (deprecated) (no effect: the value is never read) |

The `on(...)` overloads build `EventOptions(ignoreCancelled)` themselves; explicit options only reach `Events.on(type, options, handler)`.

## Priority does nothing

Every script handler is registered at one fixed slot, `EventPriority.SCRIPT` — after the client's `BEFORE_ALL` and `BEFORE` modules, before its `NOW`, `AFTER` and `AFTER_ALL` ones; among scripts the order is registration order.
The enum is `@NoEffect` since API 2.

| Constant | Description |
|---|---|
| `FIRST` | (no effect on dispatch order) |
| `EARLY` | (no effect on dispatch order) |
| `NORMAL` | default in `EventOptions` (no effect on dispatch order) |
| `LATE` | (no effect on dispatch order) |
| `LAST` | (no effect on dispatch order) |

The members that still take or return a `Priority` — the two deprecated `on(priority, ...)` overloads and `EventOptions.priority(...)` — compile and discard the value.

## Unsubscribing

| Method | Type | Description |
|---|---|---|
| `Subscription.unsubscribe()` | `void` | removes the handler; idempotent |
| `Subscription.active()` | `boolean` | true while subscribed and the script is loaded |
| `Subscription.close()` | `void` | calls `unsubscribe()`; `Subscription` is `AutoCloseable` |

Switching the script off unsubscribes every handler; switching it on registers them again.

## Threads and budget

Every event fires on the Minecraft client thread except `PacketReceiveEvent` (netty IO thread) and `PacketSendEvent` (the thread that sends the packet) — see [Packets](../actions/packets.md).
One handler invocation has 250 ms; overrunning it switches the script off, and so do 5 consecutive throws — [Sandbox and limits](../extras/limits.md#budgets).
`Render2DEvent`, `Render3DEvent`, `PacketReceiveEvent` and `PacketSendEvent` ignore `EventOptions` entirely — both `priority` and `ignoreCancelled`.
