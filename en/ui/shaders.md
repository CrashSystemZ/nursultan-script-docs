# Shaders

A shader is a small program that works out the colour of every pixel. It is what you reach for when shapes and images cannot give you the effect: waves, glows, noise, gradients driven by a formula.

## Compiling one

Write it once, at the top level of the script:

```kotlin
val glow = shader("glow", """
    #version 150

    uniform vec2 uSize;
    uniform float uTime;

    in vec2 texCoord;
    out vec4 fragColor;

    void main() {
        vec2 uv = gl_FragCoord.xy / uSize;
        float pulse = 0.5 + 0.5 * sin(uTime * 2.0);
        fragColor = vec4(0.35, 0.78, 1.0, pulse * 0.6);
    }
""")
```

The first argument is a name, which is how the shader shows up in logs. The second is the fragment shader source in GLSL. That is all you need — the vertex shader is optional and there is a standard one behind it.

## Checking it built

The shader is not built by the time `shader(...)` returns — it is compiled the first time you actually draw with it. On the line straight after compiling, `ready()` is therefore still `false` and `error()` is still `null`. Ask from inside the render handler, and say it once:

```kotlin
var reported = false

on<Render2DEvent> { e ->
    val failure = glow.error()
    if (failure != null && !reported) {
        reported = true
        log.error("shader did not build: $failure")
    }
    e.render().shader(glow, 8f, 8f, 160f, 40f)
}
```

Keep drawing either way. A shader that is never drawn is never compiled, so skipping the draw while `!ready()` leaves it broken forever; a failed one costs you nothing and simply puts no pixels on screen.

## Drawing with it

```kotlin
on<Render2DEvent> { e ->
    glow.set("uSize", e.width(), e.height())
    glow.set("uTime", client.millis() / 1000f)
    e.render().shader(glow, 8f, 8f, 160f, 40f)
}
```

`shader(...)` draws a rectangle filled by your shader.

## Passing values in

```kotlin
glow.set("uAlpha", 0.5f)
glow.set("uSize", width, height)
glow.set("uColor", 0.35f, 0.78f, 1.0f)
glow.set("uRect", x, y, w, h)
glow.set("uSteps", 8)
glow.set("uEnabled", true)                 // a bool uniform
glow.setArray("uWeights", floatArrayOf(0.2f, 0.3f, 0.5f))   // float[N]
```

The name has to match a `uniform` in the source. Values are set before drawing, in the same frame.

`setArray` is for `uniform float name[N]` — two, three and four floats already have their own `set` overloads, so it is only worth reaching for at five and up.

## Passing a texture in

`set` also takes a [texture](render-2d.md) and binds it to a `sampler2D`. You do not pick a texture unit and you do not bind anything yourself — hand over the texture, name the uniform, done:

```kotlin
head.set("u_skin", target.skinTexture() ?: return@on)
```

Several textures in one shader is just several calls; the units are handed out for you in the order you set them.

A round head is the short version of the whole idea — a shader over the skin, the face and the hat sampled from the region each of them lives in, everything outside the circle thrown away:

```kotlin
val head = shader("round-head", """
    #version 330

    in vec2 in_uv;
    out vec4 out_color;

    uniform sampler2D u_skin;
    uniform vec4 u_face;
    uniform vec4 u_hat;

    void main() {
        float dist = length(in_uv * 2.0 - 1.0);
        float aa = fwidth(dist) + 1e-5;
        float mask = 1.0 - smoothstep(1.0 - aa, 1.0 + aa, dist);
        if (mask <= 0.0) {
            discard;
        }

        vec4 face = texture(u_skin, u_face.xy + in_uv * u_face.zw);
        vec4 hat = texture(u_skin, u_hat.xy + in_uv * u_hat.zw);
        vec4 skin = mix(face, hat, hat.a);

        out_color = vec4(skin.rgb, skin.a * mask);
    }
""")

on<Render2DEvent> { e ->
    val skin = target.skinTexture() ?: return@on
    val w = skin.width().toFloat()
    val h = skin.height().toFloat()
    if (w <= 0f) return@on

    head.set("u_skin", skin)
    head.set("u_face", 8f / w, 8f / h, 8f / w, 8f / h)
    head.set("u_hat", 40f / w, 8f / h, 8f / w, 8f / h)
    e.render().shader(head, 8f, 8f, 32f, 32f)
}
```

`in_uv` comes from the standard vertex shader and runs `0..1` across the rectangle you asked for, which is exactly what you need to walk a region of the skin — no vertex shader of your own required.

`glId()` on a texture is there when you want to know what you are holding, but you never need it for this: `set` takes the texture, not the number.

## Your own vertex shader

The standard vertex shader takes the rectangle you asked for, puts it on the screen and hands the fragment shader the position, the UV and the colour. For almost every effect that is enough — the whole thing lives in the fragment shader.

Sometimes it is not enough: you want to move the corners apart, bend the quad, count something once per vertex instead of once per pixel. Then pass the vertex source as well:

```kotlin
val wave = shader(
    "wave",
    fragmentSource = """
        #version 330

        in vec2 v_uv;
        out vec4 out_color;

        void main() {
            out_color = vec4(v_uv, 1.0, 1.0);
        }
    """,
    vertexSource = """
        #version 330

        layout(location = 0) in vec3 pos;
        layout(location = 1) in vec2 uv;
        layout(location = 2) in vec4 color;

        out vec2 v_uv;

        uniform mat4 u_projection;
        uniform mat4 u_view;
        uniform float u_time;

        void main() {
            v_uv = uv;
            vec3 p = pos;
            p.y += sin(pos.x * 0.05 + u_time * 3.0) * 4.0;
            gl_Position = u_projection * u_view * vec4(p, 1.0);
        }
    """
)
```

The rectangle always arrives with the same three attributes:

| Location | Type | What it is |
| --- | --- | --- |
| `0` | `vec3` | corner position in screen pixels, `z` is zero |
| `1` | `vec2` | UV inside the rectangle, from `0,0` to `1,1` |
| `2` | `vec4` | vertex colour, BGRA |

Anything you declare `out` in the vertex shader you read back as an `in` of the same name and type in the fragment shader — the two sources are yours, so the names are up to you.

`u_projection` and `u_view` are the ones that put the rectangle where you asked for it. Declare them and multiply by them, or the shape lands somewhere else entirely. Alongside them the client fills in `u_time` (seconds since launch), `u_resolution` (framebuffer size) and `u_rect` (the rectangle you passed to `render.shader(...)`), and whatever you set yourself with `set(...)` — all of it is readable from both stages, so a uniform only the vertex shader needs works just as well.

If you would rather start from the standard one, this is it:

```glsl
#version 330

layout(location = 0) in vec3 pos;
layout(location = 1) in vec2 uv;
layout(location = 2) in vec4 color;

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

## Things to keep in mind

* The shader is built once, on the first draw — never call `shader(...)` inside a render handler, or you get a fresh one every frame.
* The fragment shader is required, the vertex one is not — leave it out and the standard one is used.
* A compile error takes down neither the client nor the script — nothing simply draws, and the reason is in `error()`. The message says which of the two stages it came from.
* Everything a shader draws is visible to you only.

## When you do not need one

Backdrop blur already exists — `render.blur(...)`, no shader needed. A two-colour gradient is `render.gradient(...)`. A shader is worth it when the effect really is computed per pixel.
