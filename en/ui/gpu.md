# Your own geometry

`gpu` is `client.gpu()` — script-owned vertex formats, meshes, pipelines and draw calls. Vertex positions are relative to the camera, the buffers are freed while the script is off, and every type on this page is API 2.

```kotlin
val format = gpu.format(VertexAttribute.floats(3), VertexAttribute.color())
val mesh = gpu.indexedMesh(format)
val pipeline = gpu.pipeline(myShader, DrawMode.TRIANGLES, BlendMode.ALPHA, DepthMode.TEST)
val pass = gpu.renderType(pipeline, mesh)

on<Render3DEvent> { event ->
    val verts = mesh.verts()
    val base = verts.putVec3(0f, 0f, 0f).putColor(Colors.RED).next()
    verts.putVec3(1f, 0f, 0f).putColor(Colors.GREEN).next()
    verts.putVec3(0f, 1f, 0f).putColor(Colors.BLUE).next()
    mesh.idx()?.putTriangle(base, base + 1, base + 2)

    myShader.setMat4("u_view", event.viewMatrix())
    myShader.setMat4("u_projection", event.projectionMatrix())
    pass.draw()
}
```

## The factory

| Method | Type | Description |
|---|---|---|
| `gpu.format(attributes...)` | `VertexFormat` | interleaved vertex layout (throws `IllegalArgumentException` below 1 or above 16 attributes) |
| `gpu.mesh(format)` | `Mesh` | non-indexed mesh, 1024-byte initial vertex buffer (throws `IllegalArgumentException` on a foreign format) |
| `gpu.indexedMesh(format)` | `Mesh` | mesh with an index buffer, 1024 bytes each |
| `gpu.pipeline(shader, drawMode, blend, depth)` | `Pipeline` | culling disabled |
| `gpu.pipeline(shader, drawMode, blend, depth, cull)` | `Pipeline` | nulls default to `TRIANGLES`/`ALPHA`/`OFF` (throws `IllegalArgumentException` on a foreign shader) |
| `gpu.renderType(pipeline, mesh)` | `RenderType` | non-instanced draw pairing |
| `gpu.instancedRenderType(pipeline, mesh, verticesPerInstance)` | `RenderType` | `verticesPerInstance` clamped to 0 or more (throws `IllegalArgumentException` on a foreign pipeline or mesh) |

Every method throws `IllegalStateException` once the script session is closed. Meshes, pipelines and render types are tracked resources; formats are not.

## The vertex format

| Method | Type | Description |
|---|---|---|
| `VertexAttribute(count, type, normalized, instanced)` | `VertexAttribute` | canonical constructor |
| `attribute.count()` | `int` | component count |
| `attribute.type()` | `AttributeType` | component type |
| `attribute.normalized()` | `boolean` | normalization flag, ignored — only `UNSIGNED_BYTE` is normalized |
| `attribute.instanced()` | `boolean` | true when the attribute advances once per instance |
| `VertexAttribute.floats(count)` | `VertexAttribute` | static, `count` floats, per-vertex |
| `VertexAttribute.ints(count)` | `VertexAttribute` | static, `count` ints, per-vertex |
| `VertexAttribute.color()` | `VertexAttribute` | static, 4 normalized unsigned bytes, per-vertex |
| `attribute.perInstance()` | `VertexAttribute` | copy with `instanced = true` |

### AttributeType

| Constant | Description |
|---|---|
| `AttributeType.FLOAT` | 32-bit float components |
| `AttributeType.INT` | 32-bit signed integer components |
| `AttributeType.UNSIGNED_BYTE` | 4 normalized bytes regardless of the declared count |

### The format itself

| Method | Type | Description |
|---|---|---|
| `format.stride()` | `int` | bytes per vertex |
| `format.attributeCount()` | `int` | attributes in the layout |

Attributes map to the shader's `layout(location = N)` inputs in declaration order; nothing checks that what you write matches the format.

## Writing vertices

| Method | Type | Description |
|---|---|---|
| `mesh.format()` | `VertexFormat` | the format the mesh was created with |
| `mesh.verts()` | `VertexWriter` | vertex writer (throws `IllegalStateException` when the mesh is closed) |
| `mesh.idx()` | `IndexWriter?` | index writer, null on a non-indexed mesh |
| `mesh.indexed()` | `boolean` | true when created by `gpu.indexedMesh` |
| `mesh.clear()` | `void` | resets both writers, no-op if the mesh was never used |

The GL buffer is created on first use, and that first write must happen on the render thread. `draw()` uploads both writers and empties them, so each frame writes its geometry from scratch.

### The vertex writer

| Method | Type | Description |
|---|---|---|
| `verts.putFloat(value)` | `VertexWriter` | writes 4 bytes |
| `verts.putInt(value)` | `VertexWriter` | writes 4 bytes |
| `verts.putVec2(x, y)` | `VertexWriter` | writes two floats, identical to `putUv` |
| `verts.putVec3(x, y, z)` | `VertexWriter` | writes three floats |
| `verts.putVec3(position)` | `VertexWriter` | same from a [`Vec`](../game/math.md), components cast to float |
| `verts.putUv(u, v)` | `VertexWriter` | writes two floats |
| `verts.putColor(argb)` | `VertexWriter` | writes the ARGB int; reaches GLSL as BGRA |
| `verts.next()` | `int` | ends the vertex, returns its 0-based index (throws `IllegalStateException` at 1048576 vertices) |
| `verts.vertexCount()` | `int` | vertices finished so far, 0 when the mesh is closed |

## Indices

| Method | Type | Description |
|---|---|---|
| `idx.put(index)` | `IndexWriter` | appends one vertex index (throws `IllegalStateException` at 4194304 indices) |
| `idx.putTriangle(a, b, c)` | `IndexWriter` | appends three indices |
| `idx.putQuad(a, b, c, d)` | `IndexWriter` | appends triangles `(a, b, c)` and `(a, c, d)` |
| `idx.indexCount()` | `int` | indices written so far, 0 when the mesh is closed |

## The pipeline

| Method | Type | Description |
|---|---|---|
| `pipeline.shader()` | `Shader` | the shader this pipeline draws with — [Shaders](shaders.md) |
| `pipeline.drawMode()` | `DrawMode` | primitive topology |

### DrawMode

| Constant | Description |
|---|---|
| `DrawMode.TRIANGLES` | `GL_TRIANGLES`, also the fallback for a null mode |
| `DrawMode.TRIANGLE_STRIP` | `GL_TRIANGLE_STRIP` |
| `DrawMode.TRIANGLE_FAN` | `GL_TRIANGLE_FAN` |
| `DrawMode.LINES` | `GL_LINES` |
| `DrawMode.LINE_STRIP` | `GL_LINE_STRIP` |
| `DrawMode.POINTS` | `GL_POINTS` |

### BlendMode

| Constant | Description |
|---|---|
| `BlendMode.OFF` | blending disabled |
| `BlendMode.ALPHA` | `SRC_ALPHA, ONE_MINUS_SRC_ALPHA`, the fallback for a null mode |
| `BlendMode.PREMULTIPLIED` | `ONE, ONE_MINUS_SRC_ALPHA` on colour and alpha |
| `BlendMode.ADDITIVE` | `SRC_ALPHA, ONE` on colour and alpha |
| `BlendMode.INVERT` | `ONE_MINUS_DST_COLOR, ONE_MINUS_SRC_COLOR` on colour, `ONE, ZERO` on alpha, the blend of the vanilla crosshair (API 7) |

### DepthMode

| Constant | Description |
|---|---|
| `DepthMode.OFF` | depth test off, depth write off, the fallback for a null mode |
| `DepthMode.TEST` | depth test on, depth write off |
| `DepthMode.TEST_AND_WRITE` | depth test on, depth write on |

## Drawing

| Method | Type | Description |
|---|---|---|
| `pass.pipeline()` | `Pipeline` | the pipeline used |
| `pass.mesh()` | `Mesh` | the mesh used |
| `pass.draw()` | `void` | binds the client framebuffer and issues one draw call (throws `IllegalStateException` when the render type or mesh is closed) |
| `pass.drawInstanced()` | `void` | draws instances of `verticesPerInstance` vertices, `instancedRenderType` only |

No built-in uniforms are supplied, unlike `Render.shader`: `u_view`, `u_projection` and anything else are yours to set before the call. Both calls draw nothing while the mesh is empty or the shader failed to compile.

## Limits

| Limit | Value |
|---|---|
| live gpu resources per script | 64 meshes, pipelines and render types together |
| attributes per vertex format | 16 |
| vertices per mesh | 1 048 576 |
| indices per mesh | 4 194 304 |
| embedded font or PNG | 8 MiB |

The exception each limit throws: [Sandbox and limits](../extras/limits.md#resource-limits).
