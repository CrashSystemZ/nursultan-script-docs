# Entries with their own logic

An `entry(name) { }` is one option of a `selectable` or a `combo` that carries its own callbacks and event handlers. They run only while that option is selected and the script is switched on.

```kotlin
val fast = entry("Fast", true) {
    onSelect { chat.print("fast mode") }
    on<ClientTickEvent> { control.jump() }
}
val smooth = entry("Smooth")

val mode = selectable("Mode", fast, smooth)
```

## Strings or entries

| Method | Type | Description |
|---|---|---|
| `selectable("Mode", "A", "B")` | `Selectable` | options as plain strings, entries built for you |
| `selectable("Mode", entry("A") { }, entry("B") { })` | `Selectable` | options as entry objects carrying their own logic |
| `entry(name, selected, body)` | `Entry` | creates one option, `selected` starts it picked (throws ScriptException when name is blank) |

`combo` takes the same two forms. A `selectable` with nothing marked selects its first option; a `combo` may start with none.
Every `selectable` and `combo` overload is listed on [Kinds of settings](types.md).

## Inside the block

| Method | Type | Description |
|---|---|---|
| `name` | `String` | display name passed to `entry()` |
| `selected` | `Boolean` | whether this option is selected right now |
| `on<E>(ignoreCancelled) { }` | `Subscription` | subscribes; the handler runs only while this option is selected |
| `on(type, ignoreCancelled) { }` | `Subscription` | same, event class given explicitly |
| `on<E>(priority, ignoreCancelled) { }` | `Subscription` | (deprecated, drop the argument) |
| `on(type, priority, ignoreCancelled) { }` | `Subscription` | (deprecated, drop the argument) |
| `onSelect { }` | `Unit` | runs when this option becomes selected |
| `onDeselect { }` | `Unit` | runs when this option stops being selected |

The subscription is registered once at load and stays registered across selection changes; deselecting filters the handler at dispatch, it does not unsubscribe.
`ignoreCancelled`, cancelling and unsubscribing behave exactly as on [Subscribing](../events/basics.md).

## The entry object

| Method | Type | Description |
|---|---|---|
| `name()` | `String` | display name; built-in entries drop the leading `entry.` |
| `id()` | `String` | locale key `entry.<scriptId>.<name>`, lowercased, spaces as dashes |
| `selected()` | `boolean` | whether this option is currently selected |
| `onSelect(action)` | `Entry` | adds an action run when selection flips to true |
| `onDeselect(action)` | `Entry` | adds an action run when selection flips to false |
| `on(type, handler)` | `Subscription` | subscribes with `EventOptions.DEFAULT` |
| `on(type, options, handler)` | `Subscription` | subscribes; runs only while selected and the script is on |

`onSelect`/`onDeselect` fire only once the entry belongs to a `selectable` or `combo`, and only on an actual change — the state at creation is never reported.
Reading and switching the selection (`value()`, `select()`, `has()`, `entry(reference)`) is on [Kinds of settings](types.md).

## Built-in module entries

`onSelect(...)`, `onDeselect(...)` and `on(...)` throw `ScriptException` on an entry that belongs to a built-in client module.
`name()`, `id()` and `selected()` work on it, and it is switched through its own `Selectable`/`Combo` — see [client modules](../extras/modules.md).
