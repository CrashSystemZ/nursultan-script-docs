# Kinds of settings

Every setting is created at the script's top level and shows up under its card in the Scripts tab in creation order. The `by` delegate reads and writes the live value.

```kotlin
val enabled by checkBox("Enabled", true)
val delay by slider("Delay", 200f, 0f, 1000f, 50f).postfix(Postfixes.MS)
val mode by selectable("Mode", "Fast", "Smooth", selected = "Fast")
val color by colorPicker("Colour", 0xFF5AC8FFL)

on<ClientTickEvent> {
    if (enabled && mode == "Fast") chat.print("$delay ms, $color")
}
```

## Creating one

| Method | Type | Description |
|---|---|---|
| `checkBox(name, value)` | `CheckBox` | boolean setting, `value` defaults to `false` |
| `checkBox(parent, name, value)` | `CheckBox` | nested under `parent` (throws ScriptException when the parent is not this script's checkbox) |
| `slider(name, value, min, max, step)` | `Slider` | float slider, `step` defaults to `1f` (throws ScriptException when min >= max, step <= 0 or value outside min..max) |
| `slider(parent, name, value, min, max, step)` | `Slider` | nested float slider, same validation |
| `slider(name, value: Int, min: Int, max: Int, step: Int)` | `Slider` | int convenience form, stored as float |
| `slider(parent, name, value: Int, min: Int, max: Int, step: Int)` | `Slider` | nested int convenience form |
| `rangeSlider(name, from, to, min, max, step)` | `RangeSlider` | two-handle slider (throws ScriptException when from..to does not fit min..max) |
| `rangeSlider(parent, name, from, to, min, max, step)` | `RangeSlider` | nested two-handle slider, same validation |
| `input(name, value, placeholder)` | `Input` | text field; `placeholder` is literal text shown while empty (API 2) |
| `input(parent, name, value, placeholder)` | `Input` | nested text field (API 2) |
| `selectable(name, vararg entries: Entry)` | `Selectable` | one-of-N from `entry()` objects, first auto-selected when none is |
| `selectable(parent, name, vararg entries: Entry)` | `Selectable` | nested one-of-N from entries |
| `selectable(name, vararg options: String, selected)` | `Selectable` | one-of-N from strings, `selected` defaults to the first option |
| `selectable(parent, name, vararg options: String, selected)` | `Selectable` | nested one-of-N from strings |
| `combo(name, vararg entries: Entry)` | `Combo` | any-of-N from `entry()` objects |
| `combo(parent, name, vararg entries: Entry)` | `Combo` | nested any-of-N from entries |
| `combo(name, vararg options: String, selected)` | `Combo` | any-of-N from strings, `selected` defaults to empty |
| `combo(parent, name, vararg options: String, selected)` | `Combo` | nested any-of-N from strings |
| `colorPicker(name, argb: Int)` | `ColorPicker` | ARGB colour, `0xAARRGGBB` |
| `colorPicker(name, argb: Long)` | `ColorPicker` | same, `Long` narrowed so a `0xFFRRGGBB` literal compiles |
| `colorPicker(parent, name, argb: Int)` | `ColorPicker` | nested ARGB colour |
| `colorPicker(parent, name, argb: Long)` | `ColorPicker` | nested, `Long` narrowed to int |
| `hotkey(name, key, action)` | `Hotkey` | key setting; `action` runs on press while the script is on |
| `hotkey(parent, name, key, action)` | `Hotkey` | nested key setting |
| `button(name, action)` | `Button` | menu button; throws inside `action` are logged, not propagated |
| `button(parent, name, action)` | `Button` | nested menu button |
| `entry(name, selected, body)` | `Entry` | one option for a `selectable` or `combo`, see [Entries](entries.md) |

Two settings with the same name under the same parent throw `IllegalArgumentException`; every creator throws `ScriptStateException` once the script is unloaded.
`hotkey` takes a `Key` from [Keys and binds](../actions/keys.md).

## The `by` delegate

| Method | Type | Description |
|---|---|---|
| `val x by checkBox(...)` | `Boolean` | reads `value()` |
| `var x by checkBox(...)` | `Boolean` | writes `value(v)` |
| `val x by slider(...)` | `Float` | reads `value()` |
| `var x by slider(...)` | `Float` | writes `value(v)`, clamped to min..max |
| `val x by rangeSlider(...)` | `ClosedFloatingPointRange<Float>` | reads `from()..to()` |
| `var x by rangeSlider(...)` | `ClosedFloatingPointRange<Float>` | writes both ends, each clamped to min..max |
| `val x by input(...)` | `String` | reads `value()` |
| `var x by input(...)` | `String` | writes `value(v)`, null becomes `""` |
| `val x by colorPicker(...)` | `Int` | reads the ARGB int, `0xAARRGGBB` |
| `var x by colorPicker(...)` | `Int` | writes the ARGB int |
| `val x by selectable(...)` | `String` | reads the selected entry's `name()` |
| `var x by selectable(...)` | `String` | selects by name or id (throws ScriptException when unknown) |
| `val x by combo(...)` | `List<String>` | reads the selected entry names, declaration order |
| `var x by combo(...)` | `List<String>` | selects exactly those names (throws ScriptException when unknown) |
| `val x by hotkey(...)` | `Key` | reads the bound key |
| `var x by hotkey(...)` | `Key` | rebinds the key |

`Button` holds no value and has no delegate.

## Common to every setting

| Method | Type | Description |
|---|---|---|
| `setting.name()` | `String` | display name |
| `setting.id()` | `String` | lookup id; the value given to `id(...)`, else the display name |
| `setting.id(stableId)` | `Setting` | sets the lookup id (script settings only) (throws ScriptException when null or blank) |
| `setting.visible()` | `boolean` | last computed visibility flag |
| `setting.visibleWhen(condition)` | `Setting` | installs a `BooleanSupplier` visibility predicate (script settings only) |

`visible()` is recomputed on every value change and every menu draw, so `visibleWhen` runs at draw rate.
On a built-in module setting `name()` returns the locale key after `.setting.`, `id()` the full locale key, and every write is marshalled onto the client thread.

## Checkbox

| Method | Type | Description |
|---|---|---|
| `checkBox.value()` | `boolean` | current checked state |
| `checkBox.value(v)` | `CheckBox` | writes the state, fires listeners |
| `checkBox.toggle()` | `CheckBox` | writes the inverted state |
| `checkBox.onChange(listener)` | `CheckBox` | `Consumer<Boolean>` fired on every write (script settings only) |
| `checkBox.id(stableId)` | `CheckBox` | covariant `Setting.id` |
| `checkBox.visibleWhen(condition)` | `CheckBox` | covariant `Setting.visibleWhen` |

`CheckBox` is the only setting usable as the `parent` of a nested setting.

## Sliders

| Method | Type | Description |
|---|---|---|
| `slider.value()` | `float` | current value, within min..max |
| `slider.intValue()` | `int` | current value rounded with `Math.round` |
| `slider.value(v)` | `Slider` | writes the value clamped to min..max, no step snapping |
| `slider.min()` | `float` | lower bound given at creation |
| `slider.max()` | `float` | upper bound given at creation |
| `slider.step()` | `float` | UI drag increment given at creation |
| `slider.postfix(unit)` | `Slider` | unit drawn after the number, null clears it (script settings only) |
| `slider.onChange(listener)` | `Slider` | `Consumer<Float>` fired on every write (script settings only) |
| `slider.id(stableId)` | `Slider` | covariant `Setting.id` |
| `slider.visibleWhen(condition)` | `Slider` | covariant `Setting.visibleWhen` |
| `rangeSlider.from()` | `float` | lower end of the selected range |
| `rangeSlider.to()` | `float` | upper end of the selected range |
| `rangeSlider.value(from, to)` | `RangeSlider` | writes both ends clamped to min..max (throws IllegalArgumentException when from > to) |
| `rangeSlider.min()` | `float` | lower bound given at creation |
| `rangeSlider.max()` | `float` | upper bound given at creation |
| `rangeSlider.step()` | `float` | UI drag increment given at creation |
| `rangeSlider.postfix(unit)` | `RangeSlider` | unit drawn after the numbers, null clears it (script settings only) |
| `rangeSlider.onChange(listener)` | `RangeSlider` | `BiConsumer<Float, Float>` receiving from and to (script settings only) |
| `rangeSlider.id(stableId)` | `RangeSlider` | covariant `Setting.id` |
| `rangeSlider.visibleWhen(condition)` | `RangeSlider` | covariant `Setting.visibleWhen` |

`step()` only drives dragging in the menu; `value(...)` is never snapped to it.

## Text and colour

| Method | Type | Description |
|---|---|---|
| `input.value()` | `String` | current text; a script input accepts any text, newlines included |
| `input.value(v)` | `Input` | writes the text, null becomes `""` |
| `input.onChange(listener)` | `Input` | `Consumer<String>` fired on every write (script settings only) |
| `input.id(stableId)` | `Input` | covariant `Setting.id` |
| `input.visibleWhen(condition)` | `Input` | covariant `Setting.visibleWhen` |
| `colorPicker.value()` | `int` | colour packed as `0xAARRGGBB` |
| `colorPicker.value(argb)` | `ColorPicker` | writes the colour, no range checks |
| `colorPicker.onChange(listener)` | `ColorPicker` | `Consumer<Integer>` fired on every write (script settings only) |
| `colorPicker.id(stableId)` | `ColorPicker` | covariant `Setting.id` |
| `colorPicker.visibleWhen(condition)` | `ColorPicker` | covariant `Setting.visibleWhen` |

On a built-in module input, text the client rejects reverts to the client default.
`Colors.rgb`, `Colors.rgba` and the rest of the ARGB packing helpers are on [2D render](../ui/render-2d.md).

## Choices

| Method | Type | Description |
|---|---|---|
| `selectable.value()` | `Entry` | currently selected entry |
| `selectable.select(entry)` | `Selectable` | selects it, deselects the rest (throws ScriptException when the entry is not this setting's) |
| `selectable.select(reference)` | `Selectable` | resolves the reference then selects (throws ScriptException when unknown) |
| `selectable.selected(entry)` | `boolean` | whether that entry is the selected one; false for foreign objects |
| `selectable.selected(reference)` | `boolean` | same by reference (throws ScriptException when unknown) |
| `selectable.entry(reference)` | `Entry` | resolves by full id, name or last id segment (throws ScriptException when unknown) |
| `selectable.entries()` | `List<Entry>` | all options, declaration order, immutable copy |
| `selectable.options()` | `List<String>` | entry display names, declaration order |
| `selectable.onChange(listener)` | `Selectable` | `Consumer<Entry>` receiving the new entry (script settings only) |
| `selectable.id(stableId)` | `Selectable` | covariant `Setting.id` |
| `selectable.visibleWhen(condition)` | `Selectable` | covariant `Setting.visibleWhen` |
| `combo.value()` | `List<Entry>` | selected entries, declaration order, immutable copy |
| `combo.value(selected)` | `Combo` | selects exactly these, deselects the rest; null clears everything |
| `combo.has(entry)` | `boolean` | whether that entry is selected; false for foreign objects |
| `combo.has(reference)` | `boolean` | same by reference (throws ScriptException when unknown) |
| `combo.set(entry, selected)` | `Combo` | selects or deselects one entry (throws ScriptException when the entry is not this setting's) |
| `combo.set(reference, selected)` | `Combo` | resolves the reference then sets (throws ScriptException when unknown) |
| `combo.entry(reference)` | `Entry` | resolves by full id, name or last id segment (throws ScriptException when unknown) |
| `combo.entries()` | `List<Entry>` | all options, declaration order, immutable copy |
| `combo.options()` | `List<String>` | entry display names, declaration order |
| `combo.onChange(listener)` | `Combo` | `Consumer<List<Entry>>` receiving the selected list (script settings only) |
| `combo.id(stableId)` | `Combo` | covariant `Setting.id` |
| `combo.visibleWhen(condition)` | `Combo` | covariant `Setting.visibleWhen` |

Reference lookup ignores case, dots, dashes, underscores and whitespace.
`Entry` itself is documented on [Entries with their own logic](entries.md).

## Hotkey and button

| Method | Type | Description |
|---|---|---|
| `hotkey.key()` | `Key` | bound key, `Key.UNKNOWN` when unbound or unmapped |
| `hotkey.key(k)` | `Hotkey` | rebinds the key, modifiers unchanged; null becomes `UNKNOWN` |
| `hotkey.mods()` | `int` | GLFW modifier bitmask held with the key, 0 when none |
| `hotkey.bound()` | `boolean` | whether the key is not `UNKNOWN` |
| `hotkey.id(stableId)` | `Hotkey` | covariant `Setting.id` |
| `hotkey.visibleWhen(condition)` | `Hotkey` | covariant `Setting.visibleWhen` |
| `button.id(stableId)` | `Button` | covariant `Setting.id` |
| `button.visibleWhen(condition)` | `Button` | covariant `Setting.visibleWhen` |

`Button` carries no value: no `value()`, no `onChange`, no delegate — only the action given at creation.

## Units

| Constant | Description |
|---|---|
| `Postfixes.PERCENT` | drawn as `%` |
| `Postfixes.DEGREES` | drawn as `°` |
| `Postfixes.SECONDS` | drawn as `s` |
| `Postfixes.TICKS` | drawn as `t` |
| `Postfixes.BLOCKS` | drawn as `b` |
| `Postfixes.MS` | drawn as `ms` |
| `Postfixes.MULTIPLIER` | drawn as `x` |
| `Postfixes.HEALTH` | drawn as `hp` |

Only `Slider` and `RangeSlider` take a postfix; without one the number is drawn bare.
