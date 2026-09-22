# Styled text

`text` is `client.text()`. Every styling call mutates the receiver and returns it, so a `Text` is a builder and a value at the same time — `copy()` is the only way to branch.

```kotlin
chat.print(
    text.literal("[teleport]")
        .color(Colors.CYAN)
        .bold(true)
        .runCommand("/spawn")
        .hoverText("go to spawn")
)
```

## Building one

| Method | Type | Description |
|---|---|---|
| `text.literal(value)` | `Text` | literal component; null becomes `""` |
| `text.empty()` | `Text` | empty component, usable as an append root |
| `text.translatable(key, args...)` | `Text` | translation component; `Text` arguments are unwrapped (throws `ScriptException` when the key is blank) |
| `text.keybind(key)` | `Text` | renders the key bound to a vanilla keybind id (throws `ScriptException` when the key is blank) |
| `text.legacy(value)` | `Text` | parses `§` codes into styled children; null gives an empty component |
| `text.fromJson(json)` | `Text` | parses a text-component json string (throws `ScriptException` when blank or invalid) |

In `legacy`, a colour code resets the style to that colour alone, `§r` resets to no style, and every other code accumulates onto the current style. A `§` with no valid code after it stays a literal character.

## Styling

| Method | Type | Description |
|---|---|---|
| `color(argb)` | `Text` | colour; the alpha byte is masked off, RGB only |
| `bold(value)` | `Text` | bold flag |
| `italic(value)` | `Text` | italic flag |
| `underlined(value)` | `Text` | underline flag |
| `strikethrough(value)` | `Text` | strikethrough flag |
| `obfuscated(value)` | `Text` | scrambled-glyph flag |
| `font(fontId)` | `Text` | font id; a bare id gets the `minecraft:` namespace (throws `ScriptException` when blank or unparsable) |
| `append(Text)` | `Text` | appends another component; null ignored (throws `ScriptException` when it was not built by `text.*`) |
| `append(String)` | `Text` | appends a literal string; null ignored |

A child appended after a style call inherits that style, as in vanilla.

## Clicks and hovers

| Method | Type | Description |
|---|---|---|
| `insertion(value)` | `Text` | what shift-click inserts into the chat box |
| `runCommand(command)` | `Text` | click runs the command (throws `ScriptException` when blank) |
| `suggestCommand(command)` | `Text` | click fills the chat box, unsent (throws `ScriptException` when blank) |
| `openUrl(url)` | `Text` | click opens a link; http and https only (throws `ScriptException` on any other scheme) |
| `copyToClipboard(value)` | `Text` | click copies the string (throws `ScriptException` when blank) |
| `hoverText(Text)` | `Text` | show-text hover; null ignored (throws `ScriptException` when it was not built by `text.*`) |
| `hoverText(String)` | `Text` | show-text hover from a literal string; null ignored |
| `hoverItem(item)` | `Text` | show-item hover; null, empty and foreign stacks are ignored |
| `hoverEntity(entity)` | `Text` | show-entity hover with type, uuid and name (throws `ScriptStateException` when the entity left the world) |

Clicks and hovers only fire from a message that is in the chat log; text drawn through `render` is not interactive.

## Reading it back

| Method | Type | Description |
|---|---|---|
| `copy()` | `Text` | deep copy of the component and its children, safe to mutate |
| `string()` | `String` | flattened plain text, no formatting codes |
| `toJson()` | `String` | text-component json (throws `ScriptException` when encoding fails) |
| `toString()` | `String` | same value as `string()` |
