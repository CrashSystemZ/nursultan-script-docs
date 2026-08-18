# Keys and binds

`keys` is `client.keys()` and answers what is physically held right now; `bind` is the script's own toggle keybind, and `Key` is the enum both of them speak.

```kotlin
onEnable { bind.set(Key.G, KeyMods.SHIFT, BindType.TOGGLE) }

on<ClientTickEvent> {
    if (keys.inputBlocked()) return@on          // the player is typing
    if (keys.isDown(Key.LEFT_SHIFT)) {
        chat.print(bind.displayName())
    }
}
```

## Reading the keyboard

| Method | Type | Description |
|---|---|---|
| `keys.isDown(key)` | `boolean` | true while that key or mouse button is held; false for `UNKNOWN` |
| `keys.mouseDown(button)` | `boolean` | true while that GLFW mouse button is held; false outside 0..7 |
| `keys.mouseX()` | `float` | cursor x in framebuffer pixels |
| `keys.mouseY()` | `float` | cursor y in framebuffer pixels |
| `keys.cursorLocked()` | `boolean` | true while the cursor is grabbed for camera look |
| `keys.lockCursor()` | `void` | grabs the cursor, queued onto the client thread |
| `keys.unlockCursor()` | `void` | releases the cursor, queued onto the client thread |
| `keys.inputBlocked()` | `boolean` | true while chat, a sign editor, the client menu or a text field takes typing |

`isDown` polls the OS directly: it stays true while a screen is open, and it never consults the client's bind system.
`mouseX()` and `mouseY()` are in the same framebuffer pixels [2D render](../ui/render-2d.md) draws in.

## The bind

**`Bind`**

| Method | Type | Description |
|---|---|---|
| `bind.key()` | `Key` | bound key, `Key.UNKNOWN` when unbound |
| `bind.mods()` | `int` | required modifier bitmask, 0 when none |
| `bind.type()` | `BindType` | `HOLD` or `TOGGLE` |
| `bind.bound()` | `boolean` | a key other than `UNKNOWN` is set |
| `bind.visible()` | `boolean` | the bind is listed in the client's bind UI |
| `bind.active()` | `boolean` | HOLD: key held now; TOGGLE: the bound action reports itself on |
| `bind.displayName()` | `String` | label like `Ctrl+Shift+G`; just the key label when mods are 0 |
| `bind.set(key, type)` | `Bind` | rebinds with no modifiers, same as `set(key, 0, type)` |
| `bind.set(key, mods, type)` | `Bind` | rebinds key, mods and type, keeping visibility; applied on the client thread |
| `bind.clear()` | `Bind` | rebinds to `Key.UNKNOWN` with no mods, keeping the type |

**`BindType`**

| Constant | Description |
|---|---|
| `TOGGLE` | press flips the bound state on and off |
| `HOLD` | state is on only while the key is held |

Writes are queued onto the client thread, so `key()` can still report the old value on the next line; a held `HOLD` bind is force-released first, and any type that is not `HOLD` becomes `TOGGLE`.
`bind` itself and the top-level `key(...)` are on [How a script works](../start/lifecycle.md); a client module's bind is `module.bind()` on [Client modules](../extras/modules.md).

## Modifiers

**`KeyMods`**

| Method | Type | Description |
|---|---|---|
| `KeyMods.NONE` | `int` | 0, no modifiers |
| `KeyMods.SHIFT` | `int` | 1, GLFW shift bit |
| `KeyMods.CONTROL` | `int` | 2, GLFW control bit |
| `KeyMods.ALT` | `int` | 4, GLFW alt bit |
| `KeyMods.SUPER` | `int` | 8, GLFW super/windows bit |
| `KeyMods.has(mods, mod)` | `boolean` | true when any bit of `mod` is set in `mods` |
| `KeyMods.of(shift, control, alt)` | `int` | mask from three flags, never sets `SUPER` |

Binds keep only `SHIFT`, `CONTROL` and `ALT`, so `SUPER` never appears in `bind.mods()`.
A modifier key bound on its own has its own bit stripped from the mask.

## Key actions

**`KeyAction`**

| Constant | Description |
|---|---|
| `PRESS` | key went down |
| `RELEASE` | key came up |
| `REPEAT` | OS auto-repeat while the key is held |

`KeyEvent.action()` carries it, and `pressed()` / `released()` compare against `PRESS` and `RELEASE`, so a `REPEAT` is neither — see [Event list](../events/reference.md).

## Every key

| Method | Type | Description |
|---|---|---|
| `key.displayName()` | `String` | short UI label, e.g. `Ctrl`, `Num 5`, `M1` |
| `key.isUnknown()` | `boolean` | true only for `UNKNOWN` |
| `key.isMouse()` | `boolean` | true for the `MOUSE_*` constants |
| `Key.byName(name)` | `Key` | enum name, then display name, both case-insensitive; `UNKNOWN` on no match |
| `key.toString()` | `String` | returns `displayName()`, not the enum name |

129 constants, in declaration order:

| Constant | Description |
|---|---|
| `UNKNOWN` | unbound, label `None` |
| `MOUSE_1` … `MOUSE_8` | mouse buttons, labels `M1` … `M8` |
| `SPACE` | spacebar, label `Space` |
| `APOSTROPHE`, `COMMA`, `MINUS`, `PERIOD`, `SLASH`, `SEMICOLON`, `EQUAL` | punctuation, labels `'` `,` `-` `.` `/` `;` `=` |
| `DIGIT_0` … `DIGIT_9` | number row, labels `0` … `9` |
| `A` … `Z` | letter keys, labels `A` … `Z` |
| `LEFT_BRACKET`, `BACKSLASH`, `RIGHT_BRACKET`, `GRAVE_ACCENT` | labels `[` `\` `]` `` ` `` |
| `WORLD_1`, `WORLD_2` | non-US extra keys, labels `World 1`, `World 2` |
| `ESCAPE`, `ENTER`, `TAB`, `BACKSPACE` | labels `Esc`, `Enter`, `Tab`, `Backspace` |
| `INSERT`, `DELETE` | labels `Ins`, `Del` |
| `RIGHT`, `LEFT`, `DOWN`, `UP` | arrow keys, labels `Right`, `Left`, `Down`, `Up` |
| `PAGE_UP`, `PAGE_DOWN`, `HOME`, `END` | labels `PgUp`, `PgDn`, `Home`, `End` |
| `CAPS_LOCK`, `SCROLL_LOCK`, `NUM_LOCK` | lock keys, labels `Caps`, `Scroll`, `Num` |
| `PRINT_SCREEN`, `PAUSE` | labels `Print`, `Pause` |
| `F1` … `F25` | function keys, labels `F1` … `F25` |
| `KP_0` … `KP_9` | numpad digits, labels `Num 0` … `Num 9` |
| `KP_DECIMAL`, `KP_DIVIDE`, `KP_MULTIPLY`, `KP_SUBTRACT`, `KP_ADD`, `KP_ENTER`, `KP_EQUAL` | numpad keys, labels `Num .` `Num /` `Num *` `Num -` `Num +` `Num Enter` `Num =` |
| `LEFT_SHIFT`, `LEFT_CONTROL`, `LEFT_ALT`, `LEFT_SUPER` | left modifiers, labels `Shift`, `Ctrl`, `Alt`, `Win` |
| `RIGHT_SHIFT`, `RIGHT_CONTROL`, `RIGHT_ALT`, `RIGHT_SUPER` | right modifiers, labels `RShift`, `RCtrl`, `RAlt`, `RWin` |
| `MENU` | context-menu key, label `Menu` |

`Key` is also the value type of the `hotkey` setting on [Kinds of settings](../settings/types.md), and of `KeyEvent.key()`.
