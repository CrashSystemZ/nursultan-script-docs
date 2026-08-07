# Kinds of settings

Settings are declared at the top level of the script and show up under its card in the Scripts tab. Menu order is declaration order.

## The list

| Setting | How to create it | What `by` reads |
|---|---|---|
| Checkbox | `checkBox("Name", true)` | `Boolean` |
| Slider | `slider("Name", 5f, 1f, 20f, step = 1f)` | `Float` |
| Range slider | `rangeSlider("Name", 2f, 8f, 0f, 20f)` | `ClosedFloatingPointRange<Float>` |
| Text field | `input("Name", "text")` | `String` |
| One choice | `selectable("Name", "A", "B", selected = "A")` | `String` |
| Several choices | `combo("Name", "A", "B", selected = listOf("A"))` | `List<String>` |
| Colour | `colorPicker("Name", Colors.rgba(90, 200, 255, 255))` | `Int` |
| Hotkey | `hotkey("Name", Key.G) { ... }` | `Key` |
| Button | `button("Name") { ... }` | — |

The slider has an integer form: `slider("Name", 5, 1, 20)` takes `Int`, and `intValue()` gives you an `Int` back without rounding by hand.

## Reading and writing

Every setting has `value()` to read and `value(...)` to write:

```kotlin
val speed = slider("Speed", 5f, 1f, 20f)

speed.value()          // 5.0
speed.intValue()       // 5
speed.value(12f)
speed.min()            // 1.0
speed.max()            // 20.0
```

The range slider works a little differently — it has two ends:

```kotlin
val delay = rangeSlider("Delay", 2f, 5f, 0f, 20f)

delay.from()           // 2.0
delay.to()             // 5.0
delay.value(3f, 8f)
```

## Units on sliders

A slider draws a bare number. `postfix(...)` puts a unit next to it — the same units the client's own sliders use:

```kotlin
val delay by slider("Delay", 200f, 0f, 1000f, 50f).postfix(Postfixes.MS)
val range by rangeSlider("Range", 3f, 5f, 0f, 6f, 0.1f).postfix(Postfixes.BLOCKS)
```

The set is fixed — you pick a unit, you do not write the text yourself:

| Value | Drawn as |
|---|---|
| `Postfixes.PERCENT` | `%` |
| `Postfixes.DEGREES` | `°` |
| `Postfixes.SECONDS` | `s` |
| `Postfixes.TICKS` | `t` |
| `Postfixes.BLOCKS` | `b` |
| `Postfixes.MS` | `ms` |
| `Postfixes.MULTIPLIER` | `x` |
| `Postfixes.HEALTH` | `hp` |

`postfix(...)` returns the slider itself, so it chains right before `by`. Without it the number stays bare.

A hotkey is created together with its action, which fires on press while the script is on:

```kotlin
hotkey("Panic", Key.RIGHT_SHIFT) { enabled = false }
```

## Reacting to changes

```kotlin
val color = colorPicker("Colour", Colors.CYAN)

color.onChange { argb ->
    chat.print("new colour: $argb")
}
```

A range slider hands both ends to the listener; a combo hands the list of selected [entries](entries.md).

## Nesting and visibility

Any setting can live inside a checkbox and shows up under it as a sub-item:

```kotlin
val fancy = checkBox("Fancy", false)
val alpha = slider(fancy, "Alpha", 0.5f, 0f, 1f, 0.05f)
val glow = checkBox(fancy, "Glow", true)
```

And you can hide it conditionally:

```kotlin
alpha.visibleWhen { fancy.value() }
```

The condition is checked while the menu draws, so keep heavy work out of it.

## Colours

A colour is a plain `Int` in `0xAARRGGBB`. You do not have to build it by hand:

```kotlin
Colors.rgb(90, 200, 255)
Colors.rgba(90, 200, 255, 128)
Colors.WHITE
Colors.withAlpha(Colors.RED, 100)
Colors.fade(Colors.RED, 0.5f)          // dim by alpha
Colors.mix(Colors.RED, Colors.BLUE, 0.5f)
Colors.red(argb)                        // take it apart again
```
