# 2D render

`Render2DEvent` hands you a drawing surface — `e.render()`. Coordinates are framebuffer pixels, not GUI-scaled units, and every call queues a command flushed after all handlers ran, so call order is draw order.

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

## The surface

| Method | Type | Description |
|---|---|---|
| `r.width()` | `float` | framebuffer width in px, refreshed each 2D frame |
| `r.height()` | `float` | framebuffer height in px, refreshed each 2D frame |

At GUI Scale 2 a 1920×1080 window still reports `width()` = 1920. Divide by [`gameSettings.scaleFactor()`](../actions/control.md#game-settings) to get vanilla GUI units.

## Shapes

| Method | Type | Description |
|---|---|---|
| `r.rect(x, y, width, height, argb)` | `void` | filled rectangle |
| `r.roundedRect(x, y, width, height, radius, argb)` | `void` | radius clamped 0..min(width, height)/2 |
| `r.roundedRect(x, y, width, height, topLeft, topRight, bottomRight, bottomLeft, argb)` | `void` | per-corner radii, clockwise from top-left, each clamped the same way |
| `r.outline(x, y, width, height, thickness, argb)` | `void` | square border inside the rect, thickness clamped 0..min(w, h)/2 |
| `r.roundedOutline(x, y, width, height, radius, thickness, argb)` | `void` | rounded border, stroke aligned INSIDE |
| `r.roundedOutline(x, y, width, height, radius, thickness, align, argb)` | `void` | null align treated as INSIDE |
| `r.circle(centerX, centerY, radius, argb)` | `void` | filled circle, negative radius treated as 0 |
| `r.ring(centerX, centerY, radius, thickness, argb)` | `void` | thickness clamped 0..radius |
| `r.triangle(x1, y1, x2, y2, x3, y3, argb)` | `void` | filled triangle, bounding quad padded 2 px |

### StrokeAlign

| Constant | Description |
|---|---|
| `StrokeAlign.INSIDE` | stroke inside the rect edge, used when align is null |
| `StrokeAlign.CENTER` | stroke centred on the edge, rect grown by half the thickness |
| `StrokeAlign.OUTSIDE` | stroke outside the edge, rect grown by the full thickness |

The size you pass is always the shape itself; the stroke grows outwards from it.

## Gradients and blur

| Method | Type | Description |
|---|---|---|
| `r.gradient(x, y, width, height, argbFrom, argbTo, horizontal)` | `void` | horizontal true = left to right, false = top to bottom |
| `r.gradientAngle(x, y, width, height, argbFrom, argbTo, angleDegrees)` | `void` | linear gradient along (cos a, sin a) in degrees, screen Y grows downward |
| `r.radialGradient(x, y, width, height, argbCenter, argbEdge)` | `void` | gradient out of the centre, corner radius 0 |
| `r.radialGradient(x, y, width, height, radius, argbCenter, argbEdge)` | `void` | corner radius clamped 0..min(w, h)/2 |
| `r.blur(x, y, width, height, radius)` | `void` | blurs what is already drawn under the region, full opacity, square corners |
| `r.blur(x, y, width, height, radius, argb, cornerRadius)` | `void` | only the alpha byte of argb is used, one radius for all corners |
| `r.blur(x, y, width, height, radius, argb, tl, tr, bl, br)` | `void` | per-corner radii in order top-left, top-right, bottom-left, bottom-right (API 2) |

Blur reads the framebuffer at its own place in the queue, so it only blurs commands issued before it. The rect is snapped to whole pixels and the blur radius to `max(1, round(radius))` px; a width or height that rounds to 0 is skipped.

## Text

| Method | Type | Description |
|---|---|---|
| `r.text(text, x, y, sizePx, argb)` | `void` | font `inter`, weight REGULAR |
| `r.text(text, x, y, sizePx, argb, font)` | `void` | named family, weight REGULAR |
| `r.text(text, x, y, sizePx, argb, weight)` | `void` | font `inter` at that weight (API 2) |
| `r.text(text, x, y, sizePx, argb, font, weight)` | `void` | named family at that weight (API 2) |
| `r.textShadow(text, x, y, sizePx, argb, shadowArgb)` | `void` | shadow copy queued at +1, +1 px before the text |
| `r.textShadow(text, x, y, sizePx, argb, shadowArgb, font)` | `void` | named family, weight REGULAR |
| `r.textShadow(text, x, y, sizePx, argb, shadowArgb, weight)` | `void` | font `inter` at that weight (API 2) |
| `r.textShadow(text, x, y, sizePx, argb, shadowArgb, font, weight)` | `void` | named family at that weight (API 2) |

`y` is the line top; the baseline sits at `y + textAscent(...)`. An unknown family falls back to `inter`, and null or empty text is skipped.

## Measuring text

| Method | Type | Description |
|---|---|---|
| `r.textWidth(text, sizePx)` | `float` | advance width in px, font `inter`, REGULAR |
| `r.textWidth(text, sizePx, font)` | `float` | width in px for the named family |
| `r.textWidth(text, sizePx, weight)` | `float` | width in px, font `inter` (API 2) |
| `r.textWidth(text, sizePx, font, weight)` | `float` | 0 for null or empty text and for an unresolvable family (API 2) |
| `r.textHeight(sizePx)` | `float` | line height in px, font `inter`, REGULAR |
| `r.textHeight(sizePx, font)` | `float` | line height in px for the named family |
| `r.textHeight(sizePx, weight)` | `float` | line height in px, font `inter` (API 2) |
| `r.textHeight(sizePx, font, weight)` | `float` | single-line height, independent of any string (API 2) |
| `r.textAscent(sizePx)` | `float` | ascent above the baseline in px, font `inter`, REGULAR |
| `r.textAscent(sizePx, font)` | `float` | ascent in px for the named family |
| `r.textAscent(sizePx, weight)` | `float` | ascent in px, font `inter` (API 2) |
| `r.textAscent(sizePx, font, weight)` | `float` | 0 when no family resolves (API 2) |
| `r.textDescent(sizePx)` | `float` | line height minus ascent in px, font `inter`, REGULAR |
| `r.textDescent(sizePx, font)` | `float` | descent in px for the named family |
| `r.textDescent(sizePx, weight)` | `float` | descent in px, font `inter` (API 2) |
| `r.textDescent(sizePx, font, weight)` | `float` | 0 when no family resolves (API 2) |

`textHeight` equals `textAscent + textDescent` at the same size, family and weight.

## Fonts and weights

| Method | Type | Description |
|---|---|---|
| `font(name, ttfFileInAssetsFolder)` | `void` | registers a TTF from `scripts/assets` under that family name |
| `font(name, ttf)` | `void` | registers a family from TTF bytes, ignored when the name is taken, the array is empty or over 8 MiB (API 2) |
| `client.fonts().register(name, ttfFileInAssetsFolder)` | `void` | what `font(name, file)` calls, applied on the next render frame |
| `client.fonts().register(name, ttf)` | `void` | what `font(name, bytes)` calls, the array is cloned (API 2) |
| `client.fonts().registered(name)` | `boolean` | true when a live family has that name, a queued font is not visible yet |

A name that already exists — including the client's own `inter`, `jetbrains-mono` and `minecraft` — is ignored, and script families are dropped when the script unloads. The byte form pairs with `base64(...)`: see [assets inside the script](../extras/assets.md#assets-inside-the-script).

### Weight

| Constant | Description |
|---|---|
| `Weight.THIN` | `wght` 100 (API 2) |
| `Weight.EXTRA_LIGHT` | `wght` 200 (API 2) |
| `Weight.LIGHT` | `wght` 300 (API 2) |
| `Weight.REGULAR` | `wght` 400, used when the weight argument is omitted or null (API 2) |
| `Weight.MEDIUM` | `wght` 500 (API 2) |
| `Weight.SEMI_BOLD` | `wght` 600 (API 2) |
| `Weight.BOLD` | `wght` 700 (API 2) |
| `Weight.EXTRA_BOLD` | `wght` 800 (API 2) |
| `Weight.BLACK` | `wght` 900 (API 2) |

## Items, heads, icons

| Method | Type | Description |
|---|---|---|
| `r.item(item, x, y, sizePx)` | `void` | item stack icon, foreign `Item` implementations ignored |
| `r.item(itemId, x, y, sizePx)` | `void` | namespaced id, malformed id ignored, unknown id resolves to air |
| `r.head(player, x, y, sizePx)` | `void` | player skin head quad, no-op for sizePx ≤ 0, StreamerMode or a missing skin |
| `r.effectIcon(effectId, x, y, sizePx)` | `void` | status-effect sprite `mob_effect/<path>`, x/y/size floored to ints |

## Textures

| Method | Type | Description |
|---|---|---|
| `r.texture(identifier, x, y, width, height)` | `void` | Minecraft resource identifier, full 0..1 UV, malformed identifier ignored |
| `r.texture(texture, x, y, width, height)` | `void` | full 0..1 UV, skipped while the GL id is 0 |
| `r.texture(texture, x, y, width, height, u0, v0, u1, v1)` | `void` | explicit UV rectangle 0..1, u0/v0 top-left, v grows downward |
| `r.image(fileInAssetsFolder, x, y, width, height)` | `void` | PNG from `scripts/assets`, decoded and uploaded once |
| `texture(identifier)` | `Texture?` | handle for a Minecraft resource id, null when malformed or empty |
| `image(fileInAssetsFolder)` | `Texture?` | handle for a PNG in `scripts/assets`, null when rejected, missing or empty |
| `image(name, png)` | `Texture?` | handle for PNG bytes, null for a blank name, empty bytes or over 8 MiB (API 2) |
| `client.textures().resource(identifier)` | `Texture?` | what `texture(identifier)` calls |
| `client.textures().image(fileInAssetsFolder)` | `Texture?` | what `image(file)` calls, uploaded on first draw |
| `client.textures().image(name, png)` | `Texture?` | what `image(name, bytes)` calls, released when the script is unloaded (API 2) |
| `handle.name()` | `String` | resource identifier string or the image name |
| `handle.glId()` | `int` | OpenGL texture id, 0 off the client main thread or before upload |
| `handle.ready()` | `boolean` | true when `glId()` is non-zero |
| `handle.width()` | `int` | mip-0 width in px, 0 off the main thread or when unavailable |
| `handle.height()` | `int` | mip-0 height in px, 0 off the main thread or when unavailable |

A file path is resolved against `scripts/assets`, with a one-time-warned fallback to the scripts root — see [assets](../extras/assets.md). `skinTexture()` on a player or a tab entry returns the same kind of handle.

## Colours

Every colour is an `int` shaped `0xAARRGGBB`.

| Method | Type | Description |
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
| `Colors.rgb(red, green, blue)` | `int` | packs opaque ARGB, channels clamped 0..255 |
| `Colors.rgba(red, green, blue, alpha)` | `int` | packs ARGB, all channels clamped 0..255 |
| `Colors.red(argb)` | `int` | red channel 0..255 |
| `Colors.green(argb)` | `int` | green channel 0..255 |
| `Colors.blue(argb)` | `int` | blue channel 0..255 |
| `Colors.alpha(argb)` | `int` | alpha channel 0..255 |
| `Colors.withAlpha(argb, alpha)` | `int` | replaces alpha clamped 0..255, keeps RGB |
| `Colors.fade(argb, factor)` | `int` | multiplies alpha by factor clamped 0..1 |
| `Colors.mix(first, second, amount)` | `int` | per-channel lerp including alpha, amount clamped 0..1 |

## A world point on the screen

| Method | Type | Description |
|---|---|---|
| `r.project(worldPosition)` | `Projection` | world position to screen pixels, rounded |
| `p.visible()` | `boolean` | false when the point projects behind the camera |
| `p.x()` | `float` | screen X in framebuffer px, 0 when not visible |
| `p.y()` | `float` | screen Y in framebuffer px, 0 when not visible |

## Clipping

| Method | Type | Description |
|---|---|---|
| `r.pushScissor(x, y, width, height)` | `void` | clips later draws to the rect, intersected with the enclosing one (API 5) |
| `r.popScissor()` | `void` | drops one clip level (API 5) (no effect: the scissor stack is empty) |

The rect is in framebuffer px, rounded and clamped to the frame; a negative width or height becomes 0.
Every scissor a render handler leaves open is dropped when that handler returns.

## Shaders

| Method | Type | Description |
|---|---|---|
| `r.shader(shader, x, y, width, height)` | `void` | draws a quad with a script shader, uniforms snapshotted at call time |

Foreign `Shader` implementations are ignored, and the quad is skipped while compilation failed. Compiling one and setting uniforms: [shaders](shaders.md).
