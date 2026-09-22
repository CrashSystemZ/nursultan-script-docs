# Шейдеры

`client.shaders().compile(name, fragmentSource)` — или короткая форма `shader(...)` на верхнем уровне — регистрирует GLSL-программу; клиент собирает её лениво, на рендер-потоке, при первой отрисовке. Униформы лежат на хэндле и снимаются снимком в момент постановки команды отрисовки; рисуют через [`render.shader(...)`](render-2d.md) или [пайплайн `gpu`](gpu.md).

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

## Компиляция

| Метод | Тип | Описание |
| --- | --- | --- |
| `shader(name, fragmentSource, vertexSource)` | `Shader` | короткая форма на верхнем уровне, `vertexSource` по умолчанию null |
| `client.shaders().compile(name, fragmentSource)` | `Shader` | только фрагментная стадия, вершинная — встроенная |
| `client.shaders().compile(name, fragmentSource, vertexSource)` | `Shader` | null или пустой `vertexSource` — берётся встроенная стадия (бросает `IllegalStateException`, когда сессия скрипта закрыта) |

Компиляция идёт на рендер-потоке при первой отрисовке, поэтому шейдер, который ни разу не рисовали, не собирается. Ошибки сборки уходят в консоль скрипта, шейдер не даёт пикселей, а все шейдеры уничтожаются при закрытии сессии скрипта.

## Проверить сборку

| Метод | Тип | Описание |
| --- | --- | --- |
| `name()` | `String` | имя, переданное в `compile` |
| `ready()` | `boolean` | true после успешной сборки и до уничтожения, false до первой отрисовки |
| `error()` | `String?` | текст ошибки компиляции с подставленными именами стадий, null до падения |

## Униформы

| Метод | Тип | Описание |
| --- | --- | --- |
| `set(uniform, float)` | `Shader` | uniform типа float |
| `set(uniform, x, y)` | `Shader` | uniform типа vec2 |
| `set(uniform, x, y, z)` | `Shader` | uniform типа vec3 |
| `set(uniform, x, y, z, w)` | `Shader` | uniform типа vec4 |
| `set(uniform, int)` | `Shader` | uniform типа int |
| `set(uniform, boolean)` | `Shader` | uniform типа int, 1 или 0 |
| `setArray(uniform, float[])` | `Shader` | длина 2/3/4 даёт vec2/vec3/vec4, остальные — массив float, null и пустой игнорируются |
| `setMat4(uniform, float[])` | `Shader` | mat4 по столбцам, игнорируется, если не ровно 16 float (API 2) |

Значения лежат на хэндле, а не заливаются сразу; null в имени униформы игнорируется. Имена, которых нет в собранной программе, запоминаются один раз и дальше пропускаются без ошибки.

## Текстуры

| Метод | Тип | Описание |
| --- | --- | --- |
| `set(uniform, texture)` | `Shader` | привязывает `sampler2D` к следующему юниту, чужой `Texture` удаляет униформу |
| `set(uniform, texture, filter, wrap)` | `Shader` | то же плюс GL-сэмплер, только на пути `gpu` render type — под `render.shader` текстура вообще не привязывается (API 2) |

### TextureFilter

| Константа | Описание |
| --- | --- |
| `TextureFilter.NEAREST` | `GL_NEAREST` на min и mag |
| `TextureFilter.LINEAR` | `GL_LINEAR` на min и mag |

### TextureWrap

| Константа | Описание |
| --- | --- |
| `TextureWrap.CLAMP` | `GL_CLAMP_TO_EDGE` по S и T |
| `TextureWrap.REPEAT` | `GL_REPEAT` по S и T |

Юниты раздаются в порядке установки текстур, начиная с 0; текстура с `glId()` равным 0 пропускается. Хэндлы `Texture` берутся из [`client.textures()`](render-2d.md).

## Встроенный вершинный шейдер

Берётся всегда, когда `vertexSource` равен null или пуст.

### Атрибуты

| Атрибут | Тип | Описание |
| --- | --- | --- |
| `pos` | `vec3` | location 0, позиция угла в пикселях фреймбуфера, z — ноль |
| `uv` | `vec2` | location 1, UV по нарисованному прямоугольнику, 0..1 |
| `color` | `vec4` | location 2, цвет вершины в порядке BGRA |

### Выходы

| Выход | Тип | Описание |
| --- | --- | --- |
| `in_pos` | `vec4` | `u_projection * u_view * vec4(pos, 1.0)` |
| `in_screen_pos` | `vec2` | `pos.xy`, пиксели фреймбуфера |
| `in_uv` | `vec2` | `uv`, 0..1 по прямоугольнику |
| `in_color` | `vec4` | `color.bgra`, то есть RGBA |

### Встроенные униформы

| Униформа | Тип | Описание |
| --- | --- | --- |
| `u_projection` | `mat4` | текущая матрица проекции |
| `u_view` | `mat4` | единичная матрица |
| `u_time` | `float` | монотонные секунды с инициализации класса |
| `u_resolution` | `vec2` | ширина и высота фреймбуфера в пикселях |
| `u_rect` | `vec4` | x, y, ширина и высота, переданные в `render.shader` |

Каждая из пяти заполняется, только если собранная программа её объявляет, и только на пути `render.shader` — [отрисовка через `gpu`](gpu.md) встроенных униформ не даёт. Фрагментная стадия читает четыре выхода выше как `in` с тем же именем и типом.

Стадия целиком:

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
