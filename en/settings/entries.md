# Entries with their own logic

The options of a `selectable` or a `combo` are not strings but **entry** objects. Strings work too, but an entry lets you attach behaviour to an option instead of comparing text.

## Strings — when there is no logic

If the options just mark a mode, strings are fine:

```kotlin
val mode by selectable("Mode", "Fast", "Smooth", selected = "Fast")

if (mode == "Fast") { ... }
```

The entries are built for you.

## Entries — when there is logic

`entry("Name")` creates an option you can hang handlers on:

```kotlin
val fast = entry("Fast", true) {
    onSelect { chat.print("fast mode") }
    onDeselect { chat.print("not fast any more") }
    on<ClientTickEvent> { control.jump() }
}

val smooth = entry("Smooth") {
    on<ClientTickEvent> {
        if (player.onGround()) {
            control.jump()
        }
    }
}

val mode = selectable("Mode", fast, smooth)
```

| Handler | When it fires |
|---|---|
| `onSelect` | the option was picked |
| `onDeselect` | another one was picked (for a `combo`, it was unchecked) |
| `on<Event>` | that event fires, but **only while this option is selected and the script is on** |

The second argument of `entry("Fast", true)` is whether it starts selected. A `selectable` needs exactly one selected option; mark none and the first one wins. A `combo` takes any number, including none.

This way a mode with its own behaviour lives in one place, and your event handler does not turn into a ladder of `if`s.

## Every event, the same as the script's

`on` inside an entry is the script's own `on` — any event, `priority` and `ignoreCancelled` included, and it hands back a `Subscription`:

```kotlin
val silent = entry("Silent", true) {
    on<MovePacketEvent>(priority = Priority.LAST) { it.onGround(true) }
    on<AttackEvent> { chat.print("hit " + it.target().name()) }
}

val legit = entry("Legit") {
    on<JumpEvent> { chat.print("jumped") }
}

val mode = selectable("Mode", silent, legit)
```

The one difference is the rule above: the handler runs only while its option is selected. Nothing is subscribed and unsubscribed as you switch options — the listener is registered once when the script loads and simply stays quiet while the option is not picked, so listing a dozen options costs nothing.

That is what makes a `selectable` a set of independent behaviours instead of a mode flag: each option subscribes to what it needs, and no one has to route events by hand.

## Working with the selection

```kotlin
mode.value()              // the selected Entry
mode.value() == fast      // identity, no strings
mode.selected(smooth)     // same thing, shorter
mode.select(fast)         // pick it
mode.entries()            // every option
mode.options()            // their names as strings
```

A combo is the same, only about a set:

```kotlin
val items = combo("Items", trident, crossbow)

items.value()             // list of selected entries
items.has(trident)
items.set(crossbow, false)
items.value(listOf(trident))
```

## Finding an entry by name

When you do not have the object — say you are driving a [client module](../extras/modules.md) — you can look one up:

```kotlin
val targets = client.modules().get("AttackAura").combo("targets")

targets.set(targets.entry("entry.players"), true)
targets.set("players", true)          // same thing, shorter
```

Lookup accepts the full key (`entry.players`), the name, or the last segment; case, dots, dashes and underscores do not matter.

## What you cannot do

You cannot attach `onSelect`, `onDeselect` or `on<Event>` to an entry of a built-in client module — it is someone else's setting, and a script may only read and switch it.
