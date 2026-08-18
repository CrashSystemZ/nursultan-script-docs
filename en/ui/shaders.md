# Shaders

`client.shaders().compile(name, fragmentSource)` — or the top-level `shader(...)` shorthand — registers a GLSL program that the client compiles lazily, on the render thread, at the first draw. Uniforms are stored on the handle and snapshotted when the draw command is queued; drawing is [`render.shader(...)`](render-2d.md) or a [`gpu` pipeline](gpu.md).

```kotlin
val glow = shader("glow", """
    #version 330
    out vec4 out_color;
    uniform float u_time;
    uniform vec3 u_color;
    uniform float u_alpha;
    void main() {
        out_color = vec4(u_color, u_alpha * (0.5 + 0.5 * sin(u_time * 2.0)));
    }
""")

on<Render2DEvent> { e ->
    glow.set("u_color", 0.35f, 0.78f, 1.0f)
    glow.set("u_alpha", 0.6f)
    e.render().shader(glow, 8f, 8f, 160f, 40f)
}
```

## Compiling

| Method | Type | Description |
| --- | --- | --- |
| `shader(name, fragmentSource, vertexSource)` | `Shader` | top-level shorthand, `vertexSource` defaults to null |
| `client.shaders().compile(name, fragmentSource)` | `Shader` | fragment stage only, built-in vertex stage |
| `client.shaders().compile(name, fragmentSource, vertexSource)` | `Shader` | null or blank `vertexSource` falls back to the built-in stage (throws `IllegalStateException` when the script session is closed) |

Compilation runs on the render thread at the first draw, so a shader that is never drawn never compiles. Compile failures are logged to the script console, the shader draws nothing, and every shader is disposed when the script session closes.

## Checking it built

| Method | Type | Description |
| --- | --- | --- |
| `name()` | `String` | name passed to `compile` |
| `ready()` | `boolean` | true once compiled and not disposed, false before the first draw |
| `error()` | `String?` | compile error with stage names substituted, null before failure |

## Uniforms

| Method | Type | Description |
| --- | --- | --- |
| `set(uniform, float)` | `Shader` | float uniform |
| `set(uniform, x, y)` | `Shader` | vec2 uniform |
| `set(uniform, x, y, z)` | `Shader` | vec3 uniform |
| `set(uniform, x, y, z, w)` | `Shader` | vec4 uniform |
| `set(uniform, int)` | `Shader` | int uniform |
| `set(uniform, boolean)` | `Shader` | int uniform, 1 or 0 |
| `setArray(uniform, float[])` | `Shader` | length 2/3/4 become vec2/vec3/vec4, other lengths a float array, null or empty ignored |
| `setMat4(uniform, float[])` | `Shader` | column-major mat4, ignored unless exactly 16 floats (API 2) |

Values are stored on the handle, not uploaded; a null `uniform` name is ignored. Names the linked program does not declare are recorded once and skipped afterwards without error.

## Textures

| Method | Type | Description |
| --- | --- | --- |
| `set(uniform, texture)` | `Shader` | binds a `sampler2D` to the next unit, a non-script `Texture` removes the uniform |
| `set(uniform, texture, filter, wrap)` | `Shader` | same plus a GL sampler object, `gpu` render-type path only — under `render.shader` the texture is not bound at all (API 2) |

### TextureFilter

| Constant | Description |
| --- | --- |
| `TextureFilter.NEAREST` | `GL_NEAREST` for min and mag |
| `TextureFilter.LINEAR` | `GL_LINEAR` for min and mag |

### TextureWrap

| Constant | Description |
| --- | --- |
| `TextureWrap.CLAMP` | `GL_CLAMP_TO_EDGE` on S and T |
| `TextureWrap.REPEAT` | `GL_REPEAT` on S and T |

Sampler units are handed out in the order the textures were set, starting at 0; a texture whose `glId()` is 0 is skipped. `Texture` handles come from [`client.textures()`](render-2d.md).

## The built-in vertex stage

Used whenever `vertexSource` is null or blank.

### Attributes

| Attribute | Type | Description |
| --- | --- | --- |
| `pos` | `vec3` | location 0, corner position in framebuffer pixels, z is 0 |
| `uv` | `vec2` | location 1, UV across the drawn rect, 0..1 |
| `color` | `vec4` | location 2, vertex colour in BGRA order |

### Outputs

| Output | Type | Description |
| --- | --- | --- |
| `in_pos` | `vec4` | `u_projection * u_view * vec4(pos, 1.0)` |
| `in_screen_pos` | `vec2` | `pos.xy`, framebuffer pixels |
| `in_uv` | `vec2` | `uv`, 0..1 across the rect |
| `in_color` | `vec4` | `color.bgra`, i.e. RGBA |

### Built-in uniforms

| Uniform | Type | Description |
| --- | --- | --- |
| `u_projection` | `mat4` | current projection matrix |
| `u_view` | `mat4` | identity matrix |
| `u_time` | `float` | monotonic seconds since class init |
| `u_resolution` | `vec2` | framebuffer width and height in pixels |
| `u_rect` | `vec4` | x, y, width and height passed to `render.shader` |

Each of the five is filled only when the linked program declares it, and only on the `render.shader` path — [`gpu` draws](gpu.md) supply no built-in uniforms. A fragment stage reads the four outputs above as `in` declarations of the same name and type.

The stage in full:

```glsl
#version 330

layout(location=0) in vec3 pos;
layout(location=1) in vec2 uv;
layout(location=2) in vec4 color;

out vec4 in_pos;
out vec2 in_screen_pos;
out vec2 in_uv;
out vec4 in_color;

uniform mat4 u_projection;
uniform mat4 u_view;

void main() {
    in_uv = uv;
    in_screen_pos = pos.xy;
    in_pos = u_projection * u_view * vec4(pos, 1.0);
    gl_Position = in_pos;
    in_color = color.bgra;
}
```
