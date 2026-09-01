# Рендер 2D

`Render2DEvent` даёт тебе поверхность для рисования — `e.render()`. Координаты — пиксели фреймбуфера, а не единицы масштаба интерфейса, и каждый вызов кладёт команду в очередь, которая сбрасывается после всех обработчиков, так что порядок вызовов и есть порядок отрисовки.

```kotlin
on<Render2DEvent> { e ->
    val r = e.render()
    val label = "fps " + client.fps()
    val w = r.textWidth(label, 10f) + 12f
    r.blur(8f, 8f, w, 22f, 12f, Colors.rgba(0, 0, 0, 160), 6f)
    r.roundedRect(8f, 8f, w, 22f, 6f, Colors.rgba(10, 12, 15, 120))
    r.text(label, 14f, 14f, 10f, Colors.WHITE)
}
```

## Поверхность

| Метод | Тип | Описание |
|---|---|---|
| `r.width()` | `float` | ширина фреймбуфера в пикселях, обновляется каждый 2D-кадр |
| `r.height()` | `float` | высота фреймбуфера в пикселях, обновляется каждый 2D-кадр |

При масштабе интерфейса 2 окно 1920×1080 всё равно даёт `width()` = 1920. Раздели на [`gameSettings.scaleFactor()`](../actions/control.md#настройки-игры), чтобы получить ванильные единицы интерфейса.

## Фигуры

| Метод | Тип | Описание |
|---|---|---|
| `r.rect(x, y, width, height, argb)` | `void` | залитый прямоугольник |
| `r.roundedRect(x, y, width, height, radius, argb)` | `void` | радиус зажат в 0..min(width, height)/2 |
| `r.roundedRect(x, y, width, height, topLeft, topRight, bottomRight, bottomLeft, argb)` | `void` | радиус на каждый угол, по часовой от левого верхнего, каждый зажат так же |
| `r.outline(x, y, width, height, thickness, argb)` | `void` | прямая рамка внутри прямоугольника, толщина зажата в 0..min(w, h)/2 |
| `r.roundedOutline(x, y, width, height, radius, thickness, argb)` | `void` | скруглённая рамка, обводка по INSIDE |
| `r.roundedOutline(x, y, width, height, radius, thickness, align, argb)` | `void` | null в align считается за INSIDE |
| `r.circle(centerX, centerY, radius, argb)` | `void` | залитый круг, отрицательный радиус считается за 0 |
| `r.ring(centerX, centerY, radius, thickness, argb)` | `void` | толщина зажата в 0..radius |
| `r.triangle(x1, y1, x2, y2, x3, y3, argb)` | `void` | залитый треугольник, габаритный квад с запасом в 2 px |

### StrokeAlign

| Константа | Описание |
|---|---|
| `StrokeAlign.INSIDE` | обводка внутри границы, берётся при null |
| `StrokeAlign.CENTER` | обводка по центру границы, прямоугольник растёт на половину толщины |
| `StrokeAlign.OUTSIDE` | обводка снаружи границы, прямоугольник растёт на всю толщину |

Размер ты всегда передаёшь у самой фигуры, обводка растёт наружу от неё.

## Градиенты и размытие

| Метод | Тип | Описание |
|---|---|---|
| `r.gradient(x, y, width, height, argbFrom, argbTo, horizontal)` | `void` | horizontal true — слева направо, false — сверху вниз |
| `r.gradientAngle(x, y, width, height, argbFrom, argbTo, angleDegrees)` | `void` | линейный градиент вдоль (cos a, sin a) в градусах, экранный Y растёт вниз |
| `r.radialGradient(x, y, width, height, argbCenter, argbEdge)` | `void` | градиент из центра, радиус углов 0 |
| `r.radialGradient(x, y, width, height, radius, argbCenter, argbEdge)` | `void` | радиус углов зажат в 0..min(w, h)/2 |
| `r.blur(x, y, width, height, radius)` | `void` | размывает то, что уже нарисовано под областью, полная непрозрачность, прямые углы |
| `r.blur(x, y, width, height, radius, argb, cornerRadius)` | `void` | из argb берётся только байт альфы, один радиус на все углы |
| `r.blur(x, y, width, height, radius, argb, tl, tr, bl, br)` | `void` | радиусы углов в порядке верхний левый, верхний правый, нижний левый, нижний правый (API 2) |

Размытие читает фреймбуфер на своём месте в очереди, поэтому размывает только команды, поставленные раньше него. Прямоугольник прижимается к целым пикселям, радиус размытия — к `max(1, round(radius))` px; ширина или высота, округлившаяся в 0, пропускается.

## Текст

| Метод | Тип | Описание |
|---|---|---|
| `r.text(text, x, y, sizePx, argb)` | `void` | шрифт `inter`, насыщенность REGULAR |
| `r.text(text, x, y, sizePx, argb, font)` | `void` | названное семейство, насыщенность REGULAR |
| `r.text(text, x, y, sizePx, argb, weight)` | `void` | шрифт `inter` с этой насыщенностью (API 2) |
| `r.text(text, x, y, sizePx, argb, font, weight)` | `void` | названное семейство с этой насыщенностью (API 2) |
| `r.textShadow(text, x, y, sizePx, argb, shadowArgb)` | `void` | копия тени ставится в очередь на +1, +1 px перед текстом |
| `r.textShadow(text, x, y, sizePx, argb, shadowArgb, font)` | `void` | названное семейство, насыщенность REGULAR |
| `r.textShadow(text, x, y, sizePx, argb, shadowArgb, weight)` | `void` | шрифт `inter` с этой насыщенностью (API 2) |
| `r.textShadow(text, x, y, sizePx, argb, shadowArgb, font, weight)` | `void` | названное семейство с этой насыщенностью (API 2) |

`y` — это верх строки, базовая линия лежит на `y + textAscent(...)`. Неизвестное семейство откатывается на `inter`, а null или пустая строка пропускается.

## Метрики текста

| Метод | Тип | Описание |
|---|---|---|
| `r.textWidth(text, sizePx)` | `float` | ширина строки в пикселях, шрифт `inter`, REGULAR |
| `r.textWidth(text, sizePx, font)` | `float` | ширина в пикселях для названного семейства |
| `r.textWidth(text, sizePx, weight)` | `float` | ширина в пикселях, шрифт `inter` (API 2) |
| `r.textWidth(text, sizePx, font, weight)` | `float` | 0 для null или пустой строки и для неизвестного семейства (API 2) |
| `r.textHeight(sizePx)` | `float` | высота строки в пикселях, шрифт `inter`, REGULAR |
| `r.textHeight(sizePx, font)` | `float` | высота строки в пикселях для названного семейства |
| `r.textHeight(sizePx, weight)` | `float` | высота строки в пикселях, шрифт `inter` (API 2) |
| `r.textHeight(sizePx, font, weight)` | `float` | высота одной строки, от самой строки не зависит (API 2) |
| `r.textAscent(sizePx)` | `float` | высота над базовой линией в пикселях, шрифт `inter`, REGULAR |
| `r.textAscent(sizePx, font)` | `float` | ascent в пикселях для названного семейства |
| `r.textAscent(sizePx, weight)` | `float` | ascent в пикселях, шрифт `inter` (API 2) |
| `r.textAscent(sizePx, font, weight)` | `float` | 0, если семейство не нашлось (API 2) |
| `r.textDescent(sizePx)` | `float` | высота строки минус ascent в пикселях, шрифт `inter`, REGULAR |
| `r.textDescent(sizePx, font)` | `float` | descent в пикселях для названного семейства |
| `r.textDescent(sizePx, weight)` | `float` | descent в пикселях, шрифт `inter` (API 2) |
| `r.textDescent(sizePx, font, weight)` | `float` | 0, если семейство не нашлось (API 2) |

`textHeight` равен `textAscent + textDescent` при том же размере, семействе и насыщенности.

## Шрифты и насыщенность

| Метод | Тип | Описание |
|---|---|---|
| `font(name, ttfFileInAssetsFolder)` | `void` | регистрирует TTF из `scripts/assets` под именем семейства |
| `font(name, ttf)` | `void` | регистрирует семейство из байтов TTF, игнорируется при занятом имени, пустом массиве и размере больше 8 МиБ (API 2) |
| `client.fonts().register(name, ttfFileInAssetsFolder)` | `void` | то, что зовёт `font(name, file)`, применяется на следующем кадре |
| `client.fonts().register(name, ttf)` | `void` | то, что зовёт `font(name, bytes)`, массив копируется (API 2) |
| `client.fonts().registered(name)` | `boolean` | true, если живое семейство с таким именем есть, шрифт в очереди ещё не виден |

Уже занятое имя — включая клиентские `inter`, `jetbrains-mono` и `minecraft` — игнорируется, а семейства скрипта снимаются при его выгрузке. Байтовая форма идёт в паре с `base64(...)`: смотри [ассеты внутри скрипта](../extras/assets.md#ассеты-внутри-скрипта).

### Weight

| Константа | Описание |
|---|---|
| `Weight.THIN` | `wght` 100 (API 2) |
| `Weight.EXTRA_LIGHT` | `wght` 200 (API 2) |
| `Weight.LIGHT` | `wght` 300 (API 2) |
| `Weight.REGULAR` | `wght` 400, берётся без аргумента и при null (API 2) |
| `Weight.MEDIUM` | `wght` 500 (API 2) |
| `Weight.SEMI_BOLD` | `wght` 600 (API 2) |
| `Weight.BOLD` | `wght` 700 (API 2) |
| `Weight.EXTRA_BOLD` | `wght` 800 (API 2) |
| `Weight.BLACK` | `wght` 900 (API 2) |

## Предметы, головы, иконки

| Метод | Тип | Описание |
|---|---|---|
| `r.item(item, x, y, sizePx)` | `void` | иконка стака, чужие реализации `Item` игнорируются |
| `r.item(itemId, x, y, sizePx)` | `void` | идентификатор с неймспейсом, кривой игнорируется, неизвестный даёт воздух |
| `r.head(player, x, y, sizePx)` | `void` | голова со скина, ничего не делает при sizePx ≤ 0, StreamerMode или отсутствии скина |
| `r.effectIcon(effectId, x, y, sizePx)` | `void` | спрайт эффекта `mob_effect/<path>`, x/y/size округляются вниз до целых |

## Текстуры

| Метод | Тип | Описание |
|---|---|---|
| `r.texture(identifier, x, y, width, height)` | `void` | идентификатор ресурса Minecraft, UV целиком 0..1, кривой игнорируется |
| `r.texture(texture, x, y, width, height)` | `void` | UV целиком 0..1, пропускается, пока GL-идентификатор равен 0 |
| `r.texture(texture, x, y, width, height, u0, v0, u1, v1)` | `void` | явный прямоугольник UV 0..1, u0/v0 — левый верхний угол, v растёт вниз |
| `r.image(fileInAssetsFolder, x, y, width, height)` | `void` | PNG из `scripts/assets`, декодируется и заливается один раз |
| `texture(identifier)` | `Texture?` | ссылка на игровую текстуру, null при кривом или пустом идентификаторе |
| `image(fileInAssetsFolder)` | `Texture?` | ссылка на PNG из `scripts/assets`, null при отказе, отсутствии или пустом файле |
| `image(name, png)` | `Texture?` | ссылка на PNG из байтов, null при пустом имени, пустых байтах или свыше 8 МиБ (API 2) |
| `client.textures().resource(identifier)` | `Texture?` | то, что зовёт `texture(identifier)` |
| `client.textures().image(fileInAssetsFolder)` | `Texture?` | то, что зовёт `image(file)`, заливается при первой отрисовке |
| `client.textures().image(name, png)` | `Texture?` | то, что зовёт `image(name, bytes)`, освобождается при выгрузке скрипта (API 2) |
| `handle.name()` | `String` | строка идентификатора ресурса или имя картинки |
| `handle.glId()` | `int` | идентификатор текстуры OpenGL, 0 вне главного потока клиента и до заливки |
| `handle.ready()` | `boolean` | true, когда `glId()` не равен 0 |
| `handle.width()` | `int` | ширина мип-уровня 0 в пикселях, 0 вне главного потока или если недоступна |
| `handle.height()` | `int` | высота мип-уровня 0 в пикселях, 0 вне главного потока или если недоступна |

Путь к файлу разрешается относительно `scripts/assets`, с откатом на корень папки скриптов и разовым предупреждением — смотри [ассеты](../extras/assets.md). `skinTexture()` у игрока или строки таба возвращает такую же ссылку.

## Цвета

Любой цвет — это `int` вида `0xAARRGGBB`.

| Метод | Тип | Описание |
|---|---|---|
| `Colors.TRANSPARENT` | `int` | `0x00000000` |
| `Colors.WHITE` | `int` | `0xFFFFFFFF` |
| `Colors.BLACK` | `int` | `0xFF000000` |
| `Colors.GRAY` | `int` | `0xFF808080` |
| `Colors.RED` | `int` | `0xFFFF5555` |
| `Colors.GREEN` | `int` | `0xFF55FF55` |
| `Colors.BLUE` | `int` | `0xFF5555FF` |
| `Colors.YELLOW` | `int` | `0xFFFFFF55` |
| `Colors.ORANGE` | `int` | `0xFFFFA500` |
| `Colors.CYAN` | `int` | `0xFF55FFFF` |
| `Colors.MAGENTA` | `int` | `0xFFFF55FF` |
| `Colors.rgb(red, green, blue)` | `int` | пакует непрозрачный ARGB, каналы зажаты в 0..255 |
| `Colors.rgba(red, green, blue, alpha)` | `int` | пакует ARGB, все каналы зажаты в 0..255 |
| `Colors.red(argb)` | `int` | канал красного 0..255 |
| `Colors.green(argb)` | `int` | канал зелёного 0..255 |
| `Colors.blue(argb)` | `int` | канал синего 0..255 |
| `Colors.alpha(argb)` | `int` | канал альфы 0..255 |
| `Colors.withAlpha(argb, alpha)` | `int` | меняет альфу с зажимом в 0..255, RGB сохраняет |
| `Colors.fade(argb, factor)` | `int` | умножает альфу на factor, зажатый в 0..1 |
| `Colors.mix(first, second, amount)` | `int` | поканальная интерполяция вместе с альфой, amount зажат в 0..1 |

## Точка мира на экране

| Метод | Тип | Описание |
|---|---|---|
| `r.project(worldPosition)` | `Projection` | мировая позиция в пиксели экрана, с округлением |
| `p.visible()` | `boolean` | false, если точка проецируется за камерой |
| `p.x()` | `float` | экранный X в пикселях фреймбуфера, 0 при невидимой точке |
| `p.y()` | `float` | экранный Y в пикселях фреймбуфера, 0 при невидимой точке |

## Отсечение

| Метод | Тип | Описание |
|---|---|---|
| `r.pushScissor(x, y, width, height)` | `void` | режет дальнейшую отрисовку по прямоугольнику, пересекая его с внешним (API 5) |
| `r.popScissor()` | `void` | снимает один уровень отсечения (API 5) (ничего не делает: стек отсечений пуст) |

Прямоугольник в пикселях фреймбуфера, округляется и зажимается в кадр; отрицательные ширина и высота становятся 0.
Каждое отсечение, оставленное обработчиком рендера открытым, снимается при его возврате.

## Смешивание

| Метод | Тип | Описание |
|---|---|---|
| `r.blend()` | `BlendMode` | режим для дальнейших фигур и текстур, ALPHA в начале обработчика (API 7) |
| `r.blend(mode)` | `void` | задаёт его для дальнейших фигур и текстур, null считается за ALPHA (API 7) |

Текст, предметы, головы, размытие и квады шейдеров смешиваются по-своему; сами режимы перечислены в [своей геометрии](gpu.md#blendmode).
`INVERT` обращает каждый пиксель под фигурой с весом её альфы — так смешивается ванильный прицел.

## Шейдеры

| Метод | Тип | Описание |
|---|---|---|
| `r.shader(shader, x, y, width, height)` | `void` | рисует квад скриптовым шейдером, униформы снимаются в момент вызова |

Чужие реализации `Shader` игнорируются, а квад пропускается, пока компиляция падает. Компиляция и униформы: [шейдеры](shaders.md).
