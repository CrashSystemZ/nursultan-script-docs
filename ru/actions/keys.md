# Клавиши и бинды

`keys` — это `client.keys()`, он отвечает, что зажато прямо сейчас; `bind` — собственный бинд-переключатель скрипта, а `Key` — перечисление, на котором говорят оба.

```kotlin
onEnable { bind.set(Key.G, KeyMods.SHIFT, BindType.TOGGLE) }

on<ClientTickEvent> {
    if (keys.inputBlocked()) return@on          // игрок печатает
    if (keys.isDown(Key.LEFT_SHIFT)) {
        chat.print(bind.displayName())
    }
}
```

## Чтение клавиатуры

| Метод | Тип | Описание |
|---|---|---|
| `keys.isDown(key)` | `boolean` | true, пока клавиша или кнопка мыши зажата; false для `UNKNOWN` |
| `keys.mouseDown(button)` | `boolean` | true, пока зажата эта кнопка мыши GLFW; false вне 0..7 |
| `keys.mouseX()` | `float` | x курсора в пикселях фреймбуфера |
| `keys.mouseY()` | `float` | y курсора в пикселях фреймбуфера |
| `keys.cursorLocked()` | `boolean` | true, пока курсор захвачен для обзора камерой |
| `keys.lockCursor()` | `void` | захватывает курсор, ставится в очередь клиентского потока |
| `keys.unlockCursor()` | `void` | отпускает курсор, ставится в очередь клиентского потока |
| `keys.inputBlocked()` | `boolean` | true, пока ввод забирают чат, редактор таблички, меню клиента или текстовое поле |

`isDown` опрашивает ОС напрямую: остаётся true и при открытом экране, и никогда не смотрит в бинд-систему клиента.
`mouseX()` и `mouseY()` — в тех же пикселях фреймбуфера, в которых рисует [Рендер 2D](../ui/render-2d.md).

## Бинд

### Bind

| Метод | Тип | Описание |
|---|---|---|
| `bind.key()` | `Key` | назначенная клавиша, `Key.UNKNOWN` если бинда нет |
| `bind.mods()` | `int` | требуемая маска модификаторов, 0 если их нет |
| `bind.type()` | `BindType` | `HOLD` или `TOGGLE` |
| `bind.bound()` | `boolean` | назначена клавиша, отличная от `UNKNOWN` |
| `bind.visible()` | `boolean` | бинд показан в списке биндов клиента |
| `bind.active()` | `boolean` | HOLD: клавиша зажата сейчас; TOGGLE: привязанное действие включено |
| `bind.displayName()` | `String` | подпись вида `Ctrl+Shift+G`; только клавиша, если маска 0 |
| `bind.set(key, type)` | `Bind` | перепривязывает без модификаторов, то же что `set(key, 0, type)` |
| `bind.set(key, mods, type)` | `Bind` | перепривязывает клавишу, маску и тип, сохраняя видимость; применяется в клиентском потоке |
| `bind.clear()` | `Bind` | ставит `Key.UNKNOWN` без модификаторов, тип сохраняется |

### BindType

| Константа | Описание |
|---|---|
| `TOGGLE` | нажатие переключает состояние вкл/выкл |
| `HOLD` | состояние включено только пока клавиша зажата |

Запись ставится в очередь клиентского потока, поэтому `key()` на следующей строке ещё может отдать старое значение; зажатый `HOLD`-бинд перед этим принудительно отпускается, а любой тип кроме `HOLD` становится `TOGGLE`.
Сам `bind` и верхнеуровневый `key(...)` — на странице [Как устроен скрипт](../start/lifecycle.md); бинд модуля клиента — это `module.bind()` на странице [Модули клиента](../extras/modules.md).

## Модификаторы

### KeyMods

| Метод | Тип | Описание |
|---|---|---|
| `KeyMods.NONE` | `int` | 0, без модификаторов |
| `KeyMods.SHIFT` | `int` | 1, бит shift в GLFW |
| `KeyMods.CONTROL` | `int` | 2, бит control в GLFW |
| `KeyMods.ALT` | `int` | 4, бит alt в GLFW |
| `KeyMods.SUPER` | `int` | 8, бит super/windows в GLFW |
| `KeyMods.has(mods, mod)` | `boolean` | true, когда в `mods` стоит хотя бы один бит `mod` |
| `KeyMods.of(shift, control, alt)` | `int` | маска из трёх флагов, `SUPER` никогда не ставит |

Бинды хранят только `SHIFT`, `CONTROL` и `ALT`, поэтому `SUPER` в `bind.mods()` не появляется никогда.
У модификатора, забинденного сам на себя, свой бит из маски вырезается.

## Фазы нажатия

### KeyAction

| Константа | Описание |
|---|---|
| `PRESS` | клавиша пошла вниз |
| `RELEASE` | клавиша отпущена |
| `REPEAT` | автоповтор ОС, пока клавиша зажата |

Её отдаёт `KeyEvent.action()`, а `pressed()` / `released()` сравнивают с `PRESS` и `RELEASE`, поэтому `REPEAT` не является ни тем, ни другим — см. [Список событий](../events/reference.md).

## Все клавиши

| Метод | Тип | Описание |
|---|---|---|
| `key.displayName()` | `String` | короткая подпись в UI, например `Ctrl`, `Num 5`, `M1` |
| `key.isUnknown()` | `boolean` | true только для `UNKNOWN` |
| `key.isMouse()` | `boolean` | true для констант `MOUSE_*` |
| `Key.byName(name)` | `Key` | сначала имя константы, потом подпись, без учёта регистра; иначе `UNKNOWN` |
| `key.toString()` | `String` | возвращает `displayName()`, а не имя константы |

### Key

129 констант, в порядке объявления:

| Константа | Описание |
|---|---|
| `UNKNOWN` | без привязки, подпись `None` |
| `MOUSE_1` … `MOUSE_8` | кнопки мыши, подписи `M1` … `M8` |
| `SPACE` | пробел, подпись `Space` |
| `APOSTROPHE`, `COMMA`, `MINUS`, `PERIOD`, `SLASH`, `SEMICOLON`, `EQUAL` | пунктуация, подписи `'` `,` `-` `.` `/` `;` `=` |
| `DIGIT_0` … `DIGIT_9` | цифровой ряд, подписи `0` … `9` |
| `A` … `Z` | буквенные клавиши, подписи `A` … `Z` |
| `LEFT_BRACKET`, `BACKSLASH`, `RIGHT_BRACKET`, `GRAVE_ACCENT` | подписи `[` `\` `]` `` ` `` |
| `WORLD_1`, `WORLD_2` | доп. клавиши не-US раскладок, подписи `World 1`, `World 2` |
| `ESCAPE`, `ENTER`, `TAB`, `BACKSPACE` | подписи `Esc`, `Enter`, `Tab`, `Backspace` |
| `INSERT`, `DELETE` | подписи `Ins`, `Del` |
| `RIGHT`, `LEFT`, `DOWN`, `UP` | стрелки, подписи `Right`, `Left`, `Down`, `Up` |
| `PAGE_UP`, `PAGE_DOWN`, `HOME`, `END` | подписи `PgUp`, `PgDn`, `Home`, `End` |
| `CAPS_LOCK`, `SCROLL_LOCK`, `NUM_LOCK` | клавиши-замки, подписи `Caps`, `Scroll`, `Num` |
| `PRINT_SCREEN`, `PAUSE` | подписи `Print`, `Pause` |
| `F1` … `F25` | функциональные клавиши, подписи `F1` … `F25` |
| `KP_0` … `KP_9` | цифры нумпада, подписи `Num 0` … `Num 9` |
| `KP_DECIMAL`, `KP_DIVIDE`, `KP_MULTIPLY`, `KP_SUBTRACT`, `KP_ADD`, `KP_ENTER`, `KP_EQUAL` | клавиши нумпада, подписи `Num .` `Num /` `Num *` `Num -` `Num +` `Num Enter` `Num =` |
| `LEFT_SHIFT`, `LEFT_CONTROL`, `LEFT_ALT`, `LEFT_SUPER` | левые модификаторы, подписи `Shift`, `Ctrl`, `Alt`, `Win` |
| `RIGHT_SHIFT`, `RIGHT_CONTROL`, `RIGHT_ALT`, `RIGHT_SUPER` | правые модификаторы, подписи `RShift`, `RCtrl`, `RAlt`, `RWin` |
| `MENU` | клавиша контекстного меню, подпись `Menu` |

`Key` — ещё и тип значения настройки `hotkey` на странице [Виды настроек](../settings/types.md), и результат `KeyEvent.key()`.
