# Your own geometry

[3D render](render-3d.md) draws lines, boxes and polygons in a fixed colour, and [shaders](shaders.md) fill a flat rectangle on top of the screen. When you need neither — a particle cloud, a textured billboard, a mesh with thousands of instances — you build the draw yourself: a vertex format, a mesh, a pipeline, and a render type that ties them together.

```kotlin
val format = gpu.format(
    VertexAttribute.floats(3),
    VertexAttribute.color()
)

val mesh = gpu.mesh(format)
val pipeline = gpu.pipeline(myShader, DrawMode.TRIANGLES, BlendMode.ALPHA, DepthMode.TEST_AND_WRITE)
val pass = gpu.renderType(pipeline, mesh)

on<Render3DEvent> { event ->
    val verts = mesh.verts()
    verts.putVec3(0f, 0f, 0f).putColor(Colors.RED).next()
    verts.putVec3(1f, 0f, 0f).putColor(Colors.GREEN).next()
    verts.putVec3(0f, 1f, 0f).putColor(Colors.BLUE).next()

    myShader.setMat4("u_view", event.viewMatrix())
    myShader.setMat4("u_projection", event.projectionMatrix())
    pass.draw()
}
```

Build the format, the mesh, the pipeline and the render type once, at the top level. Fill the mesh and draw inside the render handler — the mesh empties itself after every draw, so each frame writes its geometry from scratch.

## Positions are relative to the camera

Every coordinate you write is an offset from the camera, not a world position. Subtract `event.camera()` from whatever you have:

```kotlin
val camera = event.camera()
val target = someEntity.position()

verts.putVec3(
    (target.x() - camera.x()).toFloat(),
    (target.y() - camera.y()).toFloat(),
    (target.z() - camera.z()).toFloat()
).putColor(Colors.WHITE).next()
```

This is what keeps distant geometry from tearing apart: world coordinates are far too large for the `float` a vertex buffer holds.

## The vertex format

The format says what one record looks like. The attributes line up with the `layout(location = N)` inputs of your vertex shader, in order:

```kotlin
val format = gpu.format(
    VertexAttribute.floats(3),   // location 0
    VertexAttribute.floats(2),   // location 1
    VertexAttribute.color()      // location 2
)
```

| Attribute | What it writes | In GLSL |
|---|---|---|
| `floats(n)` | `n` floats | `vec2`, `vec3`, `vec4` |
| `ints(n)` | `n` integers | `ivec2`, `ivec3` |
| `color()` | four bytes | `vec4`, already `0…1` |

Colours arrive swizzled — write `a_color.bgra` in the shader to get RGBA back.

The writer has one method per piece, and `next()` closes the record:

```kotlin
verts.putVec3(x, y, z).putUv(u, v).putColor(argb).next()
```

Write exactly what the format describes, in the same order. Nothing checks it for you: a missed `putFloat` shifts every record after it and the picture turns to noise.

## Indices

`gpu.indexedMesh(format)` gives the mesh an index buffer, so a shared vertex is written once:

```kotlin
val mesh = gpu.indexedMesh(format)

val idx = mesh.idx() ?: return@on
val base = verts.vertexCount()
// four corners of a quad
idx.putQuad(base, base + 1, base + 2, base + 3)
```

`putQuad` writes the two triangles for you; `putTriangle` and `put` are there when you lay the geometry out yourself. `idx()` is `null` on a mesh built with `gpu.mesh(...)`.

## The pipeline

The pipeline is the shader plus the state the draw runs under:

```kotlin
val pipeline = gpu.pipeline(shader, DrawMode.TRIANGLES, BlendMode.ADDITIVE, DepthMode.TEST)
```

| | |
|---|---|
| `DrawMode` | `TRIANGLES`, `TRIANGLE_STRIP`, `TRIANGLE_FAN`, `LINES`, `LINE_STRIP`, `POINTS` |
| `BlendMode` | `OFF`, `ALPHA`, `PREMULTIPLIED`, `ADDITIVE` |
| `DepthMode` | `OFF`, `TEST`, `TEST_AND_WRITE` |

A fifth argument turns on back-face culling; leave it out and both sides of every triangle are drawn.

`DepthMode` is the one worth thinking about. `TEST_AND_WRITE` is what solid geometry wants — it hides behind blocks and behind itself. `TEST` hides behind the world but not behind other draws of your own, which is what you want for anything translucent stacked on itself, particles above all: writing depth would make them cut each other into visible squares. `OFF` draws over everything.

## Matrices

Nothing is passed to your shader on your behalf. The frame's matrices come from the event, and you set them yourself:

```kotlin
shader.setMat4("u_view", event.viewMatrix())
shader.setMat4("u_projection", event.projectionMatrix())
```

Both are sixteen floats in column-major order, ready for a `mat4`. A minimal vertex shader:

```glsl
#version 330

layout(location = 0) in vec3 a_pos;
layout(location = 1) in vec4 a_color;

out vec4 v_color;

uniform mat4 u_view;
uniform mat4 u_projection;

void main() {
    gl_Position = u_projection * u_view * vec4(a_pos, 1.0);
    v_color = a_color.bgra;
}
```

You can build a matrix yourself too — a model transform, for instance — and hand it over as sixteen floats in the same order.

## Textures

`set(uniform, texture)` binds a texture to a sampler with whatever filtering it carries, which for most of the game's own textures is nearest-neighbour. Ask for what you need instead:

```kotlin
val sprite = texture("minecraft:textures/particle/glow.png")

shader.set("u_texture", sprite, TextureFilter.LINEAR, TextureWrap.CLAMP)
```

`TextureFilter` is `NEAREST` or `LINEAR`, `TextureWrap` is `CLAMP` or `REPEAT`. The choice lives on the draw, not on the texture, so asking for linear filtering here does not change how that texture looks anywhere else in the game.

## Instancing

For many copies of one shape — particles, markers, a field of quads — write one record per copy instead of one per vertex. Mark the attributes with `perInstance()`:

```kotlin
val format = gpu.format(
    VertexAttribute.floats(3).perInstance(),   // centre
    VertexAttribute.floats(1).perInstance(),   // size
    VertexAttribute.color().perInstance()
)

val pass = gpu.instancedRenderType(pipeline, mesh, 6)
```

The last argument is how many vertices one copy has — six for a quad built from two triangles. The vertex shader gets no positions at all: it builds them from `gl_VertexID` and the per-instance data. A billboard that always faces the camera falls out of doing that in view space, where the screen axes are simply x and y:

```glsl
const vec2 CORNERS[6] = vec2[](
    vec2(-1.0, -1.0), vec2(1.0, -1.0), vec2(1.0, 1.0),
    vec2(-1.0, -1.0), vec2(1.0, 1.0), vec2(-1.0, 1.0)
);

void main() {
    vec4 viewCentre = u_view * vec4(a_centre, 1.0);
    gl_Position = u_projection * (viewCentre + vec4(CORNERS[gl_VertexID] * a_size, 0.0, 0.0));
}
```

Then `pass.drawInstanced()` instead of `pass.draw()`.

## Time, and why `millis()` will bite you

Animate with the tick counter, not with the clock:

```kotlin
val time = (client.tick() + event.tickDelta()) / 20f
```

`client.millis()` is the wall clock — around 1.8e12 today. Divided by a thousand and put in a `Float`, neighbouring values are already 128 apart, so `sin(time + phase)` returns the same number for every `phase` you can think of. Animations freeze, and a ring of particles collapses into a single point. The tick counter starts at zero when the client starts and stays small.

## Lifetime

There is no way to free any of this from a script, and none is needed. Switching the script off frees the buffers; switching it back on rebuilds them on the next frame. Reloading or removing the script releases everything, and the console says what went.

Nothing is allocated on the graphics card until the first draw, so a script that is loaded but switched off costs nothing, however much geometry it declares.

## Limits

One script gets 64 resources — meshes, pipelines and render types together — and a format takes at most 16 attributes. One mesh holds a million vertices and four million indices. Crossing any of those throws in your script, at the line that asked for too much.
