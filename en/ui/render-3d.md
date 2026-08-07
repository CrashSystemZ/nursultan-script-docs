# 3D render

The `Render3DEvent` event fires every frame and draws inside the world. It carries `render()`, `tickDelta()` and the camera position `camera()`.

Coordinates here are world coordinates, the same ones `player.position()` uses.

```kotlin
on<Render3DEvent> { e ->
    val r = e.render()
    for (target in world.players()) {
        if (target.isSelf() || target.isAlly()) continue
        r.entityBox(target, Colors.RED, true)
    }
}
```

## What you can draw

| Method | What it draws |
|---|---|
| `box(box, argb, throughWalls)` | the edges of a box |
| `filledBox(box, argb, throughWalls)` | a filled box |
| `entityBox(entity, argb, throughWalls)` | a box around an entity |
| `line(from, to, argb, throughWalls)` | a line between two points |
| `tracer(target, argb, throughWalls)` | a line from the crosshair to a point |
| `triangle(a, b, c, argb, throughWalls)` | a filled triangle |
| `quad(a, b, c, d, argb, throughWalls)` | a filled quad |
| `polygon(points, argb, throughWalls)` | a filled convex polygon |

Points are world coordinates, the same ones `player.position()` uses. A translucent disc on the ground around you:

```kotlin
on<Render3DEvent> { e ->
    val centre = player.renderPosition().add(0.0, 0.02, 0.0)
    val ring = (0 until 24).map {
        val angle = Math.toRadians(it * 15.0)
        centre.add(Math.cos(angle) * 3.0, 0.0, Math.sin(angle) * 3.0)
    }
    e.render().polygon(ring, Colors.rgba(90, 200, 255, 60), true)
}
```

`polygon` fans the points out from the first one, so it draws a **convex** shape correctly and a concave one wrongly. Points must be in order around the shape, and coplanar if you want it flat.

The last argument is the same everywhere: `true` shows through blocks, `false` hides behind them.

## Boxes

Take a ready-made box or build one:

```kotlin
entity.box()                              // an entity's box
world.block(x, y, z).box()                // a block's box
Box.around(position, 0.5)                 // a cube around a point

entity.box().expand(0.1)                  // a little bigger
```

## Tracers

A line from the middle of the screen to a target is the loudest way to show where an enemy is:

```kotlin
on<Render3DEvent> { e ->
    val enemy = world.nearestEntity(player.position(), 64.0, filters.attackable()) ?: return@on
    e.render().tracer(enemy.renderPosition().add(0.0, enemy.height() / 2.0, 0.0), Colors.RED, true)
}
```

## Smoothness

For drawing use `renderPosition()` rather than `position()`: it is already smoothed between ticks, so the box does not judder.

```kotlin
val box = entity.box().offset(entity.renderPosition().subtract(entity.position()))
```

Or simpler — `entityBox` does it for you.

## Labels above entities

Text is not drawn in the world; it lives on the HUD. To label an entity, turn its position into screen coordinates with [`project`](render-2d.md) inside `Render2DEvent`.

## Do not overload the frame

Drawing is cheaper than computing, but not free. Hundreds of boxes every frame will cost you fps. Narrow the entity list down ahead of time — by distance and filters — and keep it in a variable you refresh in `ClientTickEvent`.

```kotlin
var targets: List<Entity> = emptyList()

on<ClientTickEvent> {
    whenInGame {
        targets = world.entitiesNear(player.position(), 48.0, filters.attackable())
    }
}

on<Render3DEvent> { e ->
    for (target in targets) {
        e.render().entityBox(target, Colors.RED, true)
    }
}
```
