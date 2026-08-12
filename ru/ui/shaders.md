# Шейдеры

Шейдер — это маленькая программа, которая считает цвет каждого пикселя. Пригодится, когда фигурами и картинками нужного эффекта не собрать: волны, свечение, шум, градиенты по формуле.

## Скомпилировать

Пишется один раз на верхнем уровне скрипта:

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

Первый аргумент — имя, по нему шейдер видно в логах. Второй — исходник фрагментного шейдера на GLSL. Этого достаточно: вершинный шейдер необязателен, за ним стоит стандартный.

## Проверить, что собрался

К моменту, когда `shader(...)` вернул объект, шейдер ещё не собран — он компилируется при первой отрисовке. Поэтому строкой ниже `ready()` всё ещё `false`, а `error()` — `null`. Спрашивай из обработчика отрисовки и говори один раз:

```kotlin
var reported = false

on<Render2DEvent> { e ->
    val failure = glow.error()
    if (failure != null && !reported) {
        reported = true
        log.error("шейдер не собрался: $failure")
    }
    e.render().shader(glow, 8f, 8f, 160f, 40f)
}
```

Рисовать нужно в любом случае. Шейдер, который ни разу не рисовали, никогда не соберётся — так что выход по `!ready()` оставит его сломанным навсегда; а упавший ничего не стоит и просто не даёт пикселей.

## Нарисовать

```kotlin
on<Render2DEvent> { e ->
    glow.set("uSize", e.width(), e.height())
    glow.set("uTime", client.tick() / 20f)
    e.render().shader(glow, 8f, 8f, 160f, 40f)
}
```

`shader(...)` рисует прямоугольник, залитый твоим шейдером.

## Передать значения

```kotlin
glow.set("uAlpha", 0.5f)
glow.set("uSize", width, height)
glow.set("uColor", 0.35f, 0.78f, 1.0f)
glow.set("uRect", x, y, w, h)
glow.set("uSteps", 8)
glow.set("uEnabled", true)                 // uniform типа bool
glow.setArray("uWeights", floatArrayOf(0.2f, 0.3f, 0.5f))   // float[N]
```

Имя должно совпадать с именем `uniform` в исходнике. Значения устанавливаются перед отрисовкой, в том же кадре.

`setArray` — для `uniform float name[N]`. У двух, трёх и четырёх float'ов есть свои перегрузки `set`, так что он нужен начиная с пяти.

## Передать текстуру

`set` принимает и [текстуру](render-2d.md) — привяжет её к `sampler2D`. Текстурный юнит выбирать не надо и биндить ничего не надо: отдал текстуру, назвал юниформ, всё:

```kotlin
head.set("u_skin", target.skinTexture() ?: return@on)
```

Несколько текстур в одном шейдере — это просто несколько вызовов, юниты раздаются сами, в порядке установки.

Круглая голова — самая короткая формулировка всей идеи: шейдер поверх скина, лицо и шапка выбираются из своих областей, всё за кругом выбрасывается:

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

`in_uv` приходит из стандартного вершинного шейдера и идёт `0..1` по прямоугольнику, который ты попросил, — ровно то, что нужно, чтобы обойти область скина. Свой вершинный шейдер тут не нужен.

`glId()` у текстуры есть, если хочется знать, что именно держишь в руках, но для этого он не нужен: `set` принимает текстуру, а не число.

## Свой вершинный шейдер

Стандартный вершинный шейдер берёт прямоугольник, который ты попросил, ставит его на экран и отдаёт фрагментному позицию, UV и цвет. Почти для любого эффекта этого хватает — всё считается во фрагментном.

Иногда не хватает: надо развести углы, изогнуть прямоугольник, посчитать что-то один раз на вершину, а не на каждый пиксель. Тогда передай ещё и вершинный исходник:

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

Прямоугольник всегда приходит с одними и теми же тремя атрибутами:

| Location | Тип | Что это |
| --- | --- | --- |
| `0` | `vec3` | позиция угла в пикселях экрана, `z` — ноль |
| `1` | `vec2` | UV внутри прямоугольника, от `0,0` до `1,1` |
| `2` | `vec4` | цвет вершины, BGRA |

Всё, что объявишь `out` в вершинном, читается как `in` с тем же именем и типом во фрагментном — оба исходника твои, имена выбираешь сам.

`u_projection` и `u_view` — это то, что ставит прямоугольник туда, куда ты просил. Объяви их и умножь на них, иначе фигура окажется совсем не там. Рядом с ними клиент сам заполняет `u_time` (секунды с запуска), `u_resolution` (размер фреймбуфера) и `u_rect` (прямоугольник, переданный в `render.shader(...)`), плюс всё, что ты выставил через `set(...)`. Читать это можно из обеих стадий, так что uniform, нужный только вершинному шейдеру, работает так же.

Если удобнее начать со стандартного, вот он:

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

## Что учесть

* Шейдер собирается один раз, при первой отрисовке — не вызывай `shader(...)` внутри обработчика отрисовки, иначе каждый кадр будет новый.
* Фрагментный шейдер обязателен, вершинный — нет: не передал, значит берётся стандартный.
* Ошибка компиляции не роняет ни клиент, ни скрипт — просто ничего не нарисуется, а причина будет в `error()`. В сообщении видно, на какой из двух стадий она вылезла.
* Всё, что нарисовано шейдером, видно только тебе.

## Когда шейдер не нужен

Размытие подложки уже есть готовое — `render.blur(...)`, писать под это шейдер не надо. Двухцветный градиент — `render.gradient(...)`. Шейдер имеет смысл, когда эффект действительно считается по пикселям.
