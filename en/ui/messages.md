# Messages

`chat` prints locally and sends to the server, `log` writes to the script console, `notify` pushes a card onto the client notification stack, `clipboard` reads and writes the system clipboard. All four are roots of the script; `chat` is `client.chat()`, `log` is `client.log()`, `clipboard` is `client.clipboard()`.

```kotlin
on<ClientTickEvent> {
    if (player.health() > 6f) return@on
    chat.print(text.literal("low health").color(Colors.RED))
    notify("low health", NotifyKind.WARN)
    log.error("health " + player.health())
}
```

## Chat

| Method | Type | Description |
|---|---|---|
| `chat.print(message)` | `void` | prints `[<scriptId>] message` into your own chat only |
| `chat.printPrefixed(prefix, message)` | `void` | same with `[prefix] ` instead of the script id |
| `chat.sendToServer(message)` | `void` | sends a chat message, a leading `/` sends a server command (throws `NullPointerException` when not connected) |
| `chat.runCommand(command)` | `void` | re-sends the text with the client prefix `.` so a client command runs (throws `NullPointerException` when not connected) |

A `Text` argument to `print`/`printPrefixed` keeps its styling, anything else goes through `toString`, null prints `null` — see [Styled text](text.md).
`sendToServer` and `runCommand` ignore null and blank strings; `runCommand` strips one leading prefix character before re-adding it.

## Notifications

| Method | Type | Description |
|---|---|---|
| `notify(text, kind)` | `void` | pushes a notification card, `kind` defaults to `NotifyKind.OK` |
| `notifications.show(text)` | `void` | card with kind OK, null text becomes `""` |
| `notifications.show(text, kind)` | `void` | card with that style, null kind falls back to OK |

`notifications` is `client.notifications()`; `notify(...)` is the same call from the script root.

| Constant | Description |
|---|---|
| `OK` | success styling, the default |
| `WARN` | warning styling |
| `FAIL` | error styling |
| `ACCENT` | client accent-colour styling |

## The log

| Method | Type | Description |
|---|---|---|
| `log.info(message)` | `void` | INFO line in the script console and the client log |
| `log.warn(message)` | `void` | WARN line in the script console and the client log |
| `log.error(message)` | `void` | ERROR line in the script console and the client log |
| `log.error(message, cause)` | `void` | ERROR line plus `<file>.kts:<line>: <cause message>`, full stacktrace only in the client log |

The console is menu → **Scripts** → the terminal button; every line carries the time, the level and the script id, and it keeps the last 1000 lines.
Load, reload, toggle and compile errors are written there by the client itself, as is anything a script throws.

## Clipboard

| Method | Type | Description |
|---|---|---|
| `clipboard.get()` | `String` | clipboard text, `""` when empty or holding something that is not text (API 2) |
| `clipboard.set(text)` | `void` | replaces the clipboard, null ignored (API 2) (throws `ScriptException` above 65536 chars) |
| `clipboard.clear()` | `void` | writes an empty string (API 2) |
| `clipboard.empty()` | `boolean` | true when `get()` returns `""`, also true on a read timeout (API 2) |

Off the client thread `get()`/`empty()` hop onto it and wait up to 1000 ms, returning `""`/`true` on timeout, while `set`/`clear` are queued and do not block — see [getting onto the client thread](../extras/tasks.md#getting-onto-the-client-thread).
