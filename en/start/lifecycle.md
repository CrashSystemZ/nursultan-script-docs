# How a script works

A script is a single togglable thing, exactly like a built-in client module. It has its own bind, its own settings and its own switch in the Scripts tab.

## Loaded and on are different things

| State | What it means |
|---|---|
| **Loaded** | the file is compiled, the top level has run, settings and commands are declared |
| **On** | handlers are subscribed, commands answer, the script is working |

Loading happens when the client starts, or when you drop a file into the folder. Turning it on is the switch in the menu, or the bind.

While a script is **off** nothing of it runs: event handlers are unsubscribed and commands stop answering. Turning it on re-arms them, but the file is **not** executed again — top-level variables keep their values.

Which gives you a simple rule: reset whatever should start fresh in `onEnable`.

```kotlin
var ticks = 0
var target: Entity? = null

onEnable {
    ticks = 0
    target = null
}

onDisable {
    chat.print("off")
}
```

## Name, description, bind

```kotlin
name("AutoJump")
description("Jumps for you")
key(Key.G)
```

`key` sets the **default** bind — only if no key is assigned yet. If the player rebound it by hand, reloading the script keeps their choice.

You can read and change the bind from code too:

```kotlin
bind.key()          // which key
bind.bound()        // is anything bound at all
bind.displayName()  // how it reads in the menu
bind.type()         // BindType.TOGGLE or BindType.HOLD

bind.set(Key.R, BindType.HOLD)
bind.clear()
```

## Switching from code

`enabled` is an ordinary property:

```kotlin
enabled = false           // turn yourself off
toggle()                  // flip

hotkey("Panic", Key.RIGHT_SHIFT) { enabled = false }
```

## Reloading and unloading

Save the file and the client recompiles the script and starts a fresh instance. On/off state, the bind and setting values all come back.

`onUnload` runs when the script goes away: before a reload, when it is deleted, and when you quit. It is the place to flush something to disk:

```kotlin
onUnload {
    storage.put("runs", runs)
    storage.save()
}
```

Unsaved [configs](../settings/storage.md) are flushed for you, so you do not have to handle that case.

## API version requirements

If your script uses something new, ask for a minimum API version — on an older client it will refuse to load with a clear message instead of a confusing one:

```kotlin
requireApi(1)
```

There is a limit to what that line can do, and it is worth knowing before you rely on it: see [API versions](../extras/api-versions.md).
