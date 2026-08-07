# Styled text

A `Text` is a chat message with colour, clicks and hovers on it. You build one with `text`, and hand it wherever a message goes.

```kotlin
chat.print(text.literal("ready").color(Colors.GREEN))
```

The same type comes back out of the game: screen titles, item lore, scoreboard lines and tab rows are all `Text`, not plain strings.

## Building one

| Builder | What it makes |
|---|---|
| `text.literal("hi")` | fixed text |
| `text.empty()` | an unstyled root to hang children on |
| `text.translatable("block.minecraft.stone")` | text the client translates |
| `text.keybind("key.jump")` | the key the player has bound |
| `text.legacy("§aok")` | parses `§` colour codes into real style |
| `text.fromJson(json)` | parses the wire format |

`translatable` fills `%s` slots from the arguments you pass, and an argument can itself be a `Text`:

```kotlin
text.translatable("chat.type.text", text.literal("Notch").color(Colors.CYAN), "hello")
```

## Styling

Every styling call returns the same object, so they chain:

```kotlin
val label = text.literal("LOW HEALTH")
    .color(Colors.RED)
    .bold(true)
    .italic(false)
```

| Method | |
|---|---|
| `color(argb)` | colour; only the RGB part is used, chat has no alpha |
| `bold(v)`, `italic(v)`, `underlined(v)`, `strikethrough(v)`, `obfuscated(v)` | the vanilla toggles |
| `font(id)` | a font provider, `"minecraft:default"` and friends |
| `append(other)`, `append("text")` | add a child |
| `insertion(v)` | what shift-click inserts into the chat box |

Style set on a parent is inherited by children appended after it, exactly like vanilla:

```kotlin
val line = text.empty()
    .append(text.literal("[").color(Colors.GRAY))
    .append(text.literal("kill").color(Colors.RED).bold(true))
    .append(text.literal("] ").color(Colors.GRAY))
    .append(text.literal(target.name()))

chat.print(line)
```

## Clicks and hovers

```kotlin
chat.print(
    text.literal("[teleport]")
        .color(Colors.CYAN)
        .runCommand("/spawn")
        .hoverText("go to spawn")
)
```

| Method | What clicking does |
|---|---|
| `runCommand(cmd)` | runs it as if the player typed it |
| `suggestCommand(cmd)` | puts it in the chat box, unsent |
| `openUrl(url)` | opens a link — `http` and `https` only |
| `copyToClipboard(v)` | copies to the clipboard |

| Method | What hovering shows |
|---|---|
| `hoverText(text)` / `hoverText("...")` | a tooltip |
| `hoverItem(item)` | the item card, same as hovering an item in chat |
| `hoverEntity(entity)` | name, type and uuid |

A click event only fires from a message that is actually in chat — `chat.print` and friends. Text drawn on the HUD is a picture, nothing there is clickable.

## Reading

```kotlin
val title = game.screen()?.title ?: return
title.string()      // "Chest", styling thrown away
title.toJson()      // the wire format
title.copy()        // an independent copy you can restyle
```

`string()` is what you want for comparisons — it is the plain text a player sees:

```kotlin
if (game.screen()?.title()?.string() == "Sell") { ... }
```

## Things to keep in mind

* **A `Text` is mutable.** `color(...)` changes the object and returns it; it does not make a new one. If you keep a `Text` around and restyle it, everything holding that reference sees the change. `copy()` when that matters.
* `chat.print` takes anything — a `String`, a number, a `Text`. Only a `Text` keeps its styling.
* Colour is `0xAARRGGBB` like everywhere else in the API, but chat ignores the alpha byte.
* `text.legacy(...)` only understands `§`, not `&`. Server strings arrive with `§`; when you write the colours yourself, use the builder.
