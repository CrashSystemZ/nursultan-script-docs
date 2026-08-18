# Your first script

The file name without `.kts` is the script id; everything at the top level runs once when the file loads, and `on<...>` handlers run only while the script is switched on.

```kotlin
name("Auto jump")
key(Key.G)
val delay by slider("Delay", 5, 1, 20)
var ticks = 0
onEnable { ticks = 0 }
on<ClientTickEvent> {
    if (++ticks < delay) return@on
    ticks = 0
    whenInGame { control.jump() }
}
```

## The smallest file

| Fact | Detail |
|---|---|
| file | any `.kts` in the scripts folder, `*.gradle.kts` is ignored |
| script id | file name without `.kts`, key for configs and presets |
| empty file | compiles, appears in the Scripts tab, does nothing |
| menu label | `name(...)` when set, otherwise the script id |
| saving the file | recompiles it, restores the bind and the on/off state |

## Settings and a bind

| What | Documented on |
|---|---|
| `checkBox`, `slider`, `input`, `selectable`, `combo`, `colorPicker`, `hotkey`, `button` | [Kinds of settings](../settings/types.md) |
| `name`, `description`, `key`, `bind`, `onEnable`, `onDisable` | [How a script works](lifecycle.md) |
| `on<E> { }`, `ignoreCancelled`, `unsubscribe()` | [Subscribing](../events/basics.md) |

## Reading a setting

| Declaration | Type | Description |
|---|---|---|
| `val delay by slider("Delay", 5f, 1f, 20f)` | `Float` | delegate, reads the live value on every access |
| `val delay = slider("Delay", 5f, 1f, 20f)` | `Slider` | the setting object: `value()`, `onChange`, `visibleWhen` |

`var x by ...` writes as well: assigning to the property calls the setting's setter. The delegate type of every setting kind is on [Kinds of settings](../settings/types.md).

## Where errors go

| Destination | What lands there |
|---|---|
| script console | compile errors, handler throws as `failed - <file>.kts:<line>: <message>`, `log.info/warn/error` |
| chat | only what the script prints itself through `chat.print` |
| client log | every console line, mirrored into the Minecraft log file |

Repeated throws are throttled to one report per 3000 ms per script, and 5 throws in a row switch the script off — [Sandbox and limits](../extras/limits.md). Console and chat output are on [Messages](../ui/messages.md).
