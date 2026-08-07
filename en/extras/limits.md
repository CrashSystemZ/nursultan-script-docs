# Sandbox and limits

Scripts run in a sandbox. It is there so that somebody else's script cannot do anything unwanted to your computer.

## What you get

* The whole Nursultan API — everything this documentation covers.
* The Kotlin standard library: collections, strings, sequences, math, `Random`.
* Part of Java: collections, streams, `java.time`, `java.math`, number wrappers.

You never write imports — it is all available at the top level.

## What is not there

* Files and folders.
* The network.
* Java threads and timers.
* Reflection and class loading.
* Direct access to Minecraft and client internals.

A script that tries any of this **will not load**: the client checks it before running and tells you which symbol is blocked.

You do not need files anyway — [configs](../settings/storage.md) handle saving, and the API handles reading the world.

## How long you may think

| What | Limit |
|---|---|
| an event handler, a command, a button | 250 ms |
| running the script on load | 5 s |

Go over and the script is stopped with an error naming the file and line:

```
auto-jump.kts:14  script ran for more than 250 ms without giving the game back
```

The script switches off and the game keeps running. The client will not freeze because of someone's endless loop.

Which gives you a simple rule: no loops that might not end, and no heavy work in a single tick. Split it:

```kotlin
var index = 0

on<ClientTickEvent> {
    val all = world.entities()
    val end = minOf(index + 20, all.size)
    for (i in index until end) {
        // a little work per tick
    }
    index = if (end >= all.size) 0 else end
}
```

## Errors

Anything that throws inside a script lands in the [script console](../ui/messages.md) with the file and line. The game does not go down with it.

If a handler throws **five times in a row**, the script switches itself off so it stops burying the console in the same error. One successful call resets the counter.

## Worth remembering

* `world.entities()` every frame is bad. Once per tick, into a variable, is fine.
* Rays and prediction are computations, not field reads. Do as many as you actually need.
* Writing a config to disk is not free: `save()` does not belong in a tick handler.
* Command completions run while the player types — keep them light.

## Compatibility

A script that never touches packets survives a Minecraft update untouched. The rest of the API does not depend on the game version.

If your script needs something new, ask for the API version at the top of the file:

```kotlin
requireApi(1)
```

It does not catch everything — a call to a method the old client has never heard of fails at compile time, before that line ever runs. [API versions](api-versions.md) explains what the number does and does not buy you.
