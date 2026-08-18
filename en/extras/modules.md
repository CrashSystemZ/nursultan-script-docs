# Client modules

`client.modules()` reaches the client's own built-in modules and their flattened setting trees. Every reference — to a module, a setting or an entry — is matched with `-`, `_`, `.` and whitespace dropped and the rest lowercased, so `AttackAura`, `attack aura` and `attack-aura` all hit the same module.

```kotlin
val aura = client.modules().get("AttackAura")

chat.print(aura.displayName() + " fov " + aura.slider("fov").value())
aura.slider("fov").value(120f)
if (!aura.enabled()) {
    aura.toggle()
}
```

## Finding a module

**`Modules`**

| Method | Type | Description |
|---|---|---|
| `client.modules().all()` | `List<Module>` | every module in registry order, the order the menu shows |
| `client.modules().exists(name)` | `boolean` | true when a module matches that reference |
| `client.modules().find(name)` | `Module?` | matching module, null when nothing matches or the name is blank |
| `client.modules().get(name)` | `Module` | same lookup (throws ScriptException when nothing matches) |

Lookup tries the registry name first, then the spaced display name; the returned list is immutable and holds `DEVELOPMENT` modules only for administrator accounts.
The wrapper is cached on the module, so every lookup of the same module hands back the same `Module` instance.

## The module

**`Module`**

| Method | Type | Description |
|---|---|---|
| `module.name()` | `String` | registry name, no spaces, e.g. `AttackAura` |
| `module.displayName()` | `String` | name with spaces before capitals, e.g. `Attack Aura` |
| `module.bind()` | `Bind` | the module's toggle keybind — [Keys and binds](../actions/keys.md) |
| `module.enabled()` | `boolean` | current toggle state |
| `module.setEnabled(value)` | `void` | sets the state; ignored when it already matches |
| `module.toggle()` | `void` | flips the state |

Both writes run at once on the client thread and are queued onto it from any other thread; a module can refuse the change in its own enable/disable code.
Every state change dispatches `ModuleToggleEvent` — [Event list](../events/reference.md).

## Its settings

**`Module`**

| Method | Type | Description |
|---|---|---|
| `module.settings()` | `List<Setting>` | depth-first flattened tree, nested settings included, snapshotted at first access |
| `module.setting(reference)` | `Setting?` | by stable id, then full name, then leaf name; null when nothing matches |
| `module.checkBox(reference)` | `CheckBox` | typed lookup (throws ScriptException when missing or of another type) |
| `module.slider(reference)` | `Slider` | typed lookup (throws ScriptException when missing or of another type) |
| `module.rangeSlider(reference)` | `RangeSlider` | typed lookup (throws ScriptException when missing or of another type) |
| `module.input(reference)` | `Input` | typed lookup (throws ScriptException when missing or of another type) |
| `module.selectable(reference)` | `Selectable` | typed lookup (throws ScriptException when missing or of another type) |
| `module.combo(reference)` | `Combo` | typed lookup (throws ScriptException when missing or of another type) |
| `module.colorPicker(reference)` | `ColorPicker` | typed lookup (throws ScriptException when missing or of another type) |
| `module.hotkey(reference)` | `Hotkey` | typed lookup (throws ScriptException when missing or of another type) |

The members of those setting types are on [Kinds of settings](../settings/types.md); a group header has no script type and comes back as a plain `Setting`.
A built-in `name()` is the locale key after `.setting.` and `id()` is the whole key (`module.attackaura.setting.aim-range`); right-clicking a setting in the menu copies that key while development mode is on.

## What a module's settings refuse

| Method | Type | Description |
|---|---|---|
| `setting.id(stableId)` | `Setting` | throws ScriptException; the id is the module's locale key |
| `setting.visibleWhen(condition)` | `Setting` | throws ScriptException; the client owns menu visibility |
| `onChange(listener)` | `CheckBox` `Slider` `RangeSlider` `Input` `Selectable` `Combo` `ColorPicker` | throws ScriptException on all seven setting types |
| `postfix(unit)` | `Slider` `RangeSlider` | throws ScriptException; the unit comes from the module |
| `entry.onSelect(action)` | `Entry` | throws ScriptException on a module's entry |
| `entry.onDeselect(action)` | `Entry` | throws ScriptException on a module's entry |
| `entry.on(type, options, handler)` | `Subscription` | throws ScriptException on a module's entry |

Reads and value writes work on every built-in setting: `slider.value(v)` and `rangeSlider.value(from, to)` clamp into the module's own `min()..max()`, and each write is dispatched onto the client thread.
A module's entry is still switched through its own `Selectable`/`Combo` — [Entries with their own logic](../settings/entries.md).
