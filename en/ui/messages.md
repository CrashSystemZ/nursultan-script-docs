# Messages

## Chat

```kotlin
chat.print("hi")                           // only you, with the client prefix
chat.printPrefixed("[AutoJump]", "done")   // with your own prefix
chat.sendToServer("hi everyone")           // send to the server chat
chat.runCommand("spawn")                   // run a server command
```

`print` is seen by you alone — the server knows nothing about that message. `sendToServer` and `runCommand` really do go out.

`runCommand` does not want a leading slash: `runCommand("spawn")` sends `/spawn`.

`print` takes anything — a string, a number, or a [`Text`](text.md) when you want colour, a click or a hover:

```kotlin
chat.print(text.literal("saved").color(Colors.GREEN))
```

## Notifications

The popup in the corner — for when a message matters but chat is too wordy for it:

```kotlin
notify("done")
notify("low health", NotifyKind.WARN)
notify("did not work", NotifyKind.FAIL)
notify("target found", NotifyKind.ACCENT)
```

| Kind | What for |
|---|---|
| `OK` | ordinary, the default |
| `WARN` | a warning |
| `FAIL` | an error |
| `ACCENT` | just to stand out |

## The log

The log goes to the script console and stays out of chat. That is where debugging and anything the player does not need to see belongs:

```kotlin
log.info("loaded")
log.warn("target went away")
log.error("could not parse the answer")
log.error("it broke", exception)
```

The script's own errors land there too, together with the file and line.

## The console

Menu → **Scripts** → the terminal button next to the search field. Every line carries the time, the level and the script it came from:

```
[15:03:20] [INFO] [auto-jump] loaded
[15:03:25] [WARN] [auto-jump] target went away
[15:03:31] [ERROR] [auto-jump] failed - auto-jump.kts:12: NullPointerException
```

`[INFO]` is tinted with the client accent while its text stays white; `[WARN]` and `[ERROR]` colour the whole line so a problem is impossible to miss.

Besides your own `log` calls it shows the lifecycle the client drives itself: loaded, reloaded, removed, switched on and off, and every compile error with its message.

The pin keeps the console on screen after the menu is closed — that is the mode you want while editing a script and watching it hot-reload. The bin clears it. It holds the last 1000 lines and follows the tail on its own, unless you scroll up to read something.

## What goes where

| Where | When |
|---|---|
| `notify` | short and important: it worked, it did not |
| `chat.print` | text worth re-reading |
| `log` | debugging and detail the player does not need |
| `reply` in a command | an answer to something the player typed |

Do not write to chat every tick — that is the first reason people delete a script. If you need to show state continuously, draw it on the HUD: [2D render](render-2d.md).
