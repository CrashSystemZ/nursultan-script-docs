# Your own commands

`command("name") { }` registers a `.`-prefixed client command that lives as long as the script is switched on. Arguments are the raw tail after the node path, split on runs of whitespace.

```kotlin
command("home") {
    usage("<go|list>")
    alias("h")
    sub("go") {
        completes(0, "base", "farm")
        runs { reply("going to " + arg(0)) }
    }
}
```

## Declaring one

| Method | Type | Description |
|---|---|---|
| `command(name) { }` | `Subscription` | registers the command and its aliases (throws `ScriptException` when the name is blank, multi-word, a client command, taken by another script, or the command has neither `runs` nor `sub`) |
| `usage(text)` | `Unit` | usage text appended to missing-argument errors, null becomes `""` |
| `alias(name)` | `Unit` | extra label for the same command tree (throws `ScriptException` when blank, multi-word or taken) |
| `runs { }` | `Unit` | body run on the client thread when this node is invoked, throws are logged |
| `sub(name) { }` | `Unit` | subcommand, unlimited nesting, label lowercased to one word |
| `completes(argIndex, options)` | `Unit` | completion supplier for the 0-based argument, filtered by prefix, case-insensitive (throws `ScriptException` when argIndex < 0) |
| `completes(argIndex, vararg options)` | `Unit` | fixed completion list for that argument slot |

The name is trimmed, lowercased and loses one leading prefix character; while the script is off the command stops answering and leaves tab completion, but the name stays claimed.
A completion supplier runs while the player types; a throw inside it is logged and yields no suggestions.

## The Java builder

| Method | Type | Description |
|---|---|---|
| `client.commands().command(name)` | `CommandBuilder` | starts a definition (throws `ScriptException` when the name is empty or contains a space) |
| `client.commands().prefix()` | `String` | client command prefix as a one-char string, `.` by default |
| `usage(usage)` | `CommandBuilder` | usage text appended to missing-argument errors, null becomes `""` |
| `alias(alias)` | `CommandBuilder` | extra label for the same node (throws `ScriptException` on an empty or multi-word alias) |
| `runs(handler)` | `CommandBuilder` | handler run on the client thread (throws `ScriptException` on a null handler) |
| `sub(name, body)` | `CommandBuilder` | defines or extends a subcommand (throws `ScriptException` on an empty name or null body) |
| `completes(argIndex, options)` | `CommandBuilder` | completion supplier for the 0-based argument slot (throws `ScriptException` when argIndex < 0 or options is null) |
| `completes(argIndex, options...)` | `CommandBuilder` | fixed completion list, null array means no options |
| `register()` | `Subscription` | registers the label and every alias (throws `ScriptException` when the node has neither `runs` nor `sub`, the label is a client command, or another script owns it) |

Nothing is registered until `register()`; the Kotlin `command(name) { }` form calls it for you.
`unsubscribe()` on the returned subscription drops the label and every alias — [Subscribing](../events/basics.md).

## Inside `runs`

| Method | Type | Description |
|---|---|---|
| `label()` | `String` | full node path, e.g. `home go` |
| `args()` | `List<String>` | arguments after the node path |
| `argCount()` | `int` | number of arguments |
| `arg(index)` | `String` | 0-based argument (throws `ScriptException` naming the position and the usage text when out of range) |
| `argOr(index, fallback)` | `String` | argument, or fallback when the index is out of range |
| `intArg(index)` | `int` | argument parsed as int (throws `ScriptException` when missing or not a whole number) |
| `doubleArg(index)` | `double` | argument parsed as double (throws `ScriptException` when missing or not a number) |
| `booleanArg(index)` | `boolean` | accepts true/on/yes/1 and false/off/no/0, case-insensitive (throws `ScriptException` otherwise) |
| `rest()` | `String` | the whole argument tail as one string |
| `rest(fromIndex)` | `String` | arguments from that index joined with single spaces, whole tail when fromIndex <= 0, `""` past the end |
| `reply(message)` | `void` | prints `[<label>] message` into your own chat |
| `replyError(message)` | `void` | identical output to `reply`, no separate styling |
