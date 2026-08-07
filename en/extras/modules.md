# Client modules

`client.modules()` gives you Nursultan's built-in modules: you can read their state, switch them, and drive their settings.

## Finding a module

```kotlin
client.modules().all()             // every module
client.modules().exists("AttackAura")
client.modules().find("AttackAura")   // null when missing
client.modules().get("AttackAura")    // errors when missing
```

`find` returns `null`, `get` throws a clear error — take whichever suits the spot.

Lookup works by the internal name or by how the module reads in the menu; case, spaces and underscores do not matter:

```kotlin
client.modules().get("attack aura")
```

## What you can do with a module

```kotlin
val aura = client.modules().get("AttackAura")

aura.name()
aura.displayName()
aura.enabled()
aura.setEnabled(true)
aura.toggle()
aura.bind()
```

## Module settings

```kotlin
aura.settings()                    // every setting as a list
aura.setting("range")              // one of them, or null
```

If you know the type, ask for it directly and skip the cast:

```kotlin
aura.checkBox("...")
aura.slider("...")
aura.rangeSlider("...")
aura.input("...")
aura.selectable("...")
aura.combo("...")
aura.colorPicker("...")
aura.hotkey("...")
```

If the setting turns out to be a different type you get a clear error.

## The setting reference

Right-clicking a setting in the menu copies its reference, which looks like this:

```
module.attackaura.setting.targets.players
```

That is the most reliable way to reach a setting:

```kotlin
aura.slider("module.attackaura.setting.range").value(3.5f)
```

Shorter forms work too — the path inside the module, or the last segment:

```kotlin
aura.slider("range")
aura.combo("targets")
```

Short forms are convenient, but when a module has two settings with similar names the full reference is safer. Case, dots, dashes and underscores are ignored during lookup.

## Setting options

For a `selectable` or a `combo` the options are [entries](../settings/entries.md), and they have references too:

```kotlin
val targets = aura.combo("targets")

targets.options()                  // option names
targets.set("players", true)
targets.set(targets.entry("entry.animals"), false)

val mode = aura.selectable("mode")
mode.select("silent")
mode.value().name()
```

## What you cannot do

A module belongs to the client, not to your script. So:

* you cannot listen for changes to its settings (`onChange`);
* you cannot change its visibility (`visibleWhen`);
* you cannot change the unit on its slider (`postfix`);
* you cannot attach logic to its entries.

All of that is only for your script's own settings.

## Noticing that something was switched

```kotlin
on<ModuleToggleEvent> { e ->
    chat.print(e.name() + " -> " + e.enabled())
}
```

The event covers both client modules and scripts; `fromScript()` tells you which.
