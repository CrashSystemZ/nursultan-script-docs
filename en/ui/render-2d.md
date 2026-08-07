# 2D render

The `Render2DEvent` event fires every frame on the client overlay layer — above the game's own HUD, below the client's HUD elements. It carries `render()`, the screen size `width()` × `height()` and `tickDelta()`.

All drawing goes through `e.render()`. Coordinates are real window pixels, the same ones `width()` and `height()` are in, and they do not depend on the game's GUI Scale setting: at scale 2 a 1920×1080 screen still gives you `width()` = 1920, not 960. `project` works in that same space, so a world point can go straight next to a rectangle or a piece of text. Colours are ordinary `Int`s shaped `0xAARRGGBB`, and `Colors` is the easy way to build them.

One thing follows from that: do not hardcode sizes if you want the same look on every monitor. Place everything relative to `width()` and `height()`, and derive your font size from them too.

```kotlin
on<Render2DEvent> { e ->
    val r = e.render()
    r.roundedRect(8f, 8f, 160f, 40f, 8f, Colors.rgba(0, 0, 0, 160))
    r.text("Nursultan", 16f, 16f, 14f, Colors.WHITE)
    r.text("fps " + client.fps(), 16f, 32f, 10f, Colors.CYAN)
}
```

## Draw order

Whatever you draw later goes on top, exactly as you wrote it:

```kotlin
r.text("hidden", 10f, 10f, 10f, Colors.WHITE)
r.rect(10f, 8f, 80f, 14f, Colors.BLACK)     // covers the text

r.rect(10f, 30f, 80f, 14f, Colors.BLACK)
r.text("visible", 10f, 32f, 10f, Colors.WHITE)   // sits on the bar
```

Two things follow from that. `blur(...)` blurs everything already drawn underneath it, including your own shapes — put it before the panel it belongs to, not after. And consecutive calls of the same kind are batched into one draw, so grouping your backdrops together and your labels together is slightly cheaper than alternating them. It is a small difference; write what reads best and only regroup if you are drawing hundreds of things.

## Shapes

| Method | What it draws |
|---|---|
| `rect(x, y, w, h, argb)` | a filled rectangle |
| `roundedRect(x, y, w, h, radius, argb)` | with rounded corners |
| `roundedRect(x, y, w, h, tl, tr, br, bl, argb)` | a radius per corner, clockwise from the top left |
| `gradient(x, y, w, h, argbFrom, argbTo, horizontal)` | a two-colour gradient |
| `gradientAngle(x, y, w, h, argbFrom, argbTo, degrees)` | the same at any angle — `0` left to right, `90` top to bottom |
| `radialGradient(x, y, w, h, argbCenter, argbEdge)` | a gradient out of the centre |
| `radialGradient(x, y, w, h, radius, argbCenter, argbEdge)` | the same, with rounded corners |
| `outline(x, y, w, h, thickness, argb)` | the border only |
| `roundedOutline(x, y, w, h, radius, thickness, argb)` | a rounded border |
| `roundedOutline(x, y, w, h, radius, thickness, align, argb)` | and where the stroke sits |
| `circle(cx, cy, radius, argb)` | a circle |
| `ring(cx, cy, radius, thickness, argb)` | a ring |
| `triangle(x1, y1, x2, y2, x3, y3, argb)` | a triangle |

A chip with only its top corners rounded, and a border drawn around the outside of it:

```kotlin
r.roundedRect(8f, 8f, 120f, 24f, 7f, 7f, 0f, 0f, Colors.rgba(0, 0, 0, 160))
r.roundedOutline(8f, 8f, 120f, 24f, 7f, 1f, StrokeAlign.OUTSIDE, Colors.CYAN)
```

`StrokeAlign` decides which side of the edge the line sits on: `INSIDE` (the default, the border eats into the shape), `CENTER` (half in, half out) or `OUTSIDE` (entirely around it). The size you pass is always the shape itself — the stroke grows outwards from it, it does not shrink the rectangle.

## Blur

Blurs whatever is already drawn under the given area — that is how you make backdrops for panels:

```kotlin
r.blur(8f, 8f, 160f, 40f, 12f)
r.blur(8f, 8f, 160f, 40f, 12f, Colors.rgba(0, 0, 0, 80), 8f)
```

The second form also fills the area with a colour and rounds the corners.

## Text

```kotlin
r.text("hi", 10f, 10f, 12f, Colors.WHITE)
r.textShadow("hi", 10f, 10f, 12f, Colors.WHITE, Colors.rgba(0, 0, 0, 200))

r.textWidth("hi", 12f)
r.textHeight(12f)
r.textAscent(12f)      // from the top of the line down to the baseline
r.textDescent(12f)     // from the baseline down to the bottom of the line
```

`textHeight` is the whole line, `ascent + descent`. Use the split when you have to put something *on the baseline* — an icon next to a label, an underline, two fonts on the same line:

```kotlin
val baseline = y + r.textAscent(12f)
r.text("hi", x, y, 12f, Colors.WHITE)
r.rect(x, baseline + 1f, r.textWidth("hi", 12f), 1f, Colors.WHITE)
```

You measure width to fit a backdrop around the text:

```kotlin
val text = "health " + player.health()
val width = r.textWidth(text, 10f) + 12f
r.roundedRect(6f, 6f, width, 18f, 5f, Colors.rgba(0, 0, 0, 140))
r.text(text, 12f, 11f, 10f, Colors.WHITE)
```

## Fonts

Without a font argument you get the client's font. You can ask for the vanilla one or register your own:

```kotlin
r.text("hi", 10f, 10f, 12f, Colors.WHITE, "minecraft")

// once, at the top level of the script
font("myfont", "my-font.ttf")

// then by name
r.text("hi", 10f, 30f, 12f, Colors.WHITE, "myfont")
```

The TTF file goes into the scripts folder next to the script itself.

## Images and icons

```kotlin
r.item(inventory.held(), 10f, 10f, 16f)          // an item
r.item("minecraft:diamond", 30f, 10f, 16f)       // by id
r.head(playerEntity, 50f, 10f, 16f)              // a player's head
r.effectIcon("speed", 70f, 10f, 16f)             // an effect icon
r.texture("minecraft:textures/item/apple.png", 90f, 10f, 16f, 16f)
r.image("logo.png", 110f, 10f, 32f, 32f)         // a file from the scripts folder
```

You do not bind anything and you do not load anything: you name the texture, and it is looked up in the game's own texture manager when the frame is flushed. A name nothing answers to gets you Minecraft's missing-texture chequerboard, so a typo is loud rather than silent.

## Texture handles

Sometimes a name is not enough — you want the size, or you want to hand the thing to your own shader. Ask for the texture itself:

```kotlin
val apple = texture("minecraft:textures/item/apple.png")   // any game texture
val logo = image("logo.png")                               // a file from the scripts folder
val skin = target.skinTexture()                            // a player's skin
```

All three give you the same kind of object, and all three can be `null` — a malformed identifier, a file that is not there, a skin that has not arrived. A handle answers five questions:

```kotlin
skin.name()      // "minecraft:skins/…" — what it is called
skin.width()     // 64
skin.height()    // 64
skin.glId()      // the raw OpenGL id
skin.ready()     // has it been uploaded to the card yet
```

Everything except `name()` is only real inside a render handler — that is the only place the texture is guaranteed to be uploaded — and reads `0` / `false` anywhere else. `ready()` is worth checking before you divide by `width()`.

Handles go everywhere a name goes:

```kotlin
r.texture(skin, x, y, 32f, 32f)
```

## A piece of a texture

Give `texture` a handle and four more numbers — the corner of the source rectangle you want, `0..1` across the whole texture, top-left first:

```kotlin
r.texture(handle, x, y, width, height, u0, v0, u1, v1)
```

Cutting a piece needs the handle, not the name. That is deliberate: a name is for drawing a texture whole and forgetting about it, and once you care about regions you care about the size too.

`u0, v0` is the top-left corner, `u1, v1` the bottom-right, and `v` grows downwards like pixel rows do. Swapping a pair mirrors the piece.

A player's head is exactly that. The face sits at pixel `8,8` and is 8×8, and the hat layer that goes over it at `40,8`:

```kotlin
val skin = target.skinTexture() ?: return@on
val w = skin.width().toFloat()
val h = skin.height().toFloat()
if (w <= 0f) return@on

r.texture(skin, x, y, 32f, 32f, 8f / w, 8f / h, 16f / w, 16f / h)
r.texture(skin, x, y, 32f, 32f, 40f / w, 8f / h, 48f / w, 16f / h)
```

Two draws, the hat on top of the face — that is all `r.head(...)` does for you. Do it by hand when you want a different part of the skin, another size, or a head for someone who is not loaded in the world: [tab rows](../game/server.md) carry `skinTexture()` too, and those you have for everyone on the server. Want it round, or glowing, or greyed out — that is a [shader](shaders.md), and a texture handle is what you feed it.

## A world point on the screen

`project` turns world coordinates into screen coordinates — that is how nametags are done:

```kotlin
on<Render2DEvent> { e ->
    val r = e.render()
    for (target in world.players()) {
        if (target.isSelf()) continue
        val point = r.project(target.position().add(0.0, target.height() + 0.4, 0.0))
        if (!point.visible()) continue
        val name = target.name()
        r.text(name, point.x() - r.textWidth(name, 8f) / 2f, point.y(), 8f, Colors.WHITE)
    }
}
```

`visible()` is `false` when the point is behind you or off screen.

## Smoothness

There are more frames than ticks, so positions from `position()` will judder. For drawing use `renderPosition()`, which is already smoothed. If you compute something yourself, use `tickDelta()` from the event.

## Do not do heavy work per frame

The handler runs 60+ times a second. Do not walk every entity, shoot rays or ask for predictions inside it — compute in `ClientTickEvent`, stash it in a variable, and only draw in the frame.
