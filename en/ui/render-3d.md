# 3D render

`Render3D` comes from `Render3DEvent` and draws in world coordinates; the camera position captured when the event fired is subtracted for you. The event's own members — `render()`, `tickDelta()`, `camera()`, `viewMatrix()` and `projectionMatrix()` (API 2) — are in the [event list](../events/reference.md).

```kotlin
on<Render3DEvent> { e ->
    val targets = world.entitiesNear(player.position(), 48.0, filters.attackable())
    for (target in targets) {
        e.render().entityBox(target, Colors.RED, true)
    }
    val nearest = targets.minByOrNull { it.distanceTo(player) } ?: return@on
    e.render().tracer(nearest.renderPosition().add(0.0, nearest.height() / 2.0, 0.0), Colors.RED, true)
}
```

## What you can draw

| Method | Type | Description |
|---|---|---|
| `line(from, to, argb, throughWalls)` | `void` | single world-space line |
| `box(box, argb, throughWalls)` | `void` | the 12 wireframe edges of the box |
| `filledBox(box, argb, throughWalls)` | `void` | solid box faces |
| `entityBox(entity, argb, throughWalls)` | `void` | wireframe box from interpolated position, width and height |
| `tracer(target, argb, throughWalls)` | `void` | line from the camera near-plane centre to the point |
| `triangle(a, b, c, argb, throughWalls)` | `void` | filled triangle |
| `quad(a, b, c, d, argb, throughWalls)` | `void` | filled quad |
| `polygon(points, argb, throughWalls)` | `void` | triangle fan from the first point, no-op below 3 points |

`Vec` points and `Box` are world coordinates in blocks — see [vectors, boxes, angles](../game/math.md). `argb` is packed `0xAARRGGBB`; the constants and helpers live on [2D render](render-2d.md).

## Through walls

`throughWalls` is the last argument of every call above and picks the mesh: `true` draws into the depth-test-less one, so the shape stays visible through blocks.

## Labels

There is no world-space text. Project the world point to screen coordinates with `render.project(...)` inside `Render2DEvent` and draw the label there — see [2D render](render-2d.md).
