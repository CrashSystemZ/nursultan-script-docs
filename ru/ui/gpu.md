# Своя геометрия

[3D-рендер](render-3d.md) рисует линии, боксы и полигоны одним цветом, а [шейдеры](shaders.md) закрашивают плоский прямоугольник поверх экрана. Когда не подходит ни то, ни другое — облако частиц, текстурный биллборд, меш из тысяч копий, — отрисовку собираешь сам: формат вершины, меш, пайплайн и render type, который всё это связывает.

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

Формат, меш, пайплайн и render type создавай один раз, на верхнем уровне. Заполняй меш и рисуй внутри обработчика — после каждой отрисовки меш очищается сам, так что каждый кадр пишет геометрию заново.

## Координаты — относительно камеры

Всё, что ты пишешь в вершину, — это смещение от камеры, а не позиция в мире. Вычитай `event.camera()`:

```kotlin
val camera = event.camera()
val target = someEntity.position()

verts.putVec3(
    (target.x() - camera.x()).toFloat(),
    (target.y() - camera.y()).toFloat(),
    (target.z() - camera.z()).toFloat()
).putColor(Colors.WHITE).next()
```

Именно это не даёт далёкой геометрии рассыпаться: мировые координаты слишком велики для `float`, которым живёт вершинный буфер.

## Формат вершины

Формат описывает одну запись. Атрибуты соответствуют входам `layout(location = N)` твоего вершинного шейдера, по порядку:

```kotlin
val format = gpu.format(
    VertexAttribute.floats(3),   // location 0
    VertexAttribute.floats(2),   // location 1
    VertexAttribute.color()      // location 2
)
```

| Атрибут | Что пишет | В GLSL |
|---|---|---|
| `floats(n)` | `n` float'ов | `vec2`, `vec3`, `vec4` |
| `ints(n)` | `n` целых | `ivec2`, `ivec3` |
| `color()` | четыре байта | `vec4`, уже в диапазоне `0…1` |

Цвет приходит перемешанным — пиши в шейдере `a_color.bgra`, чтобы получить обратно RGBA.

У писателя по методу на каждую часть, а `next()` закрывает запись:

```kotlin
verts.putVec3(x, y, z).putUv(u, v).putColor(argb).next()
```

Пиши ровно то, что описано форматом, и в том же порядке. Никто это не проверяет: пропущенный `putFloat` сдвинет все записи после него, и картинка превратится в шум.

## Индексы

`gpu.indexedMesh(format)` даёт мешу индексный буфер, и общая вершина пишется один раз:

```kotlin
val mesh = gpu.indexedMesh(format)

val idx = mesh.idx() ?: return@on
val base = verts.vertexCount()
// четыре угла квада
idx.putQuad(base, base + 1, base + 2, base + 3)
```

`putQuad` сам пишет два треугольника; `putTriangle` и `put` — на случай, когда геометрию раскладываешь вручную. У меша из `gpu.mesh(...)` метод `idx()` возвращает `null`.

## Пайплайн

Пайплайн — это шейдер плюс состояние, в котором идёт отрисовка:

```kotlin
val pipeline = gpu.pipeline(shader, DrawMode.TRIANGLES, BlendMode.ADDITIVE, DepthMode.TEST)
```

| | |
|---|---|
| `DrawMode` | `TRIANGLES`, `TRIANGLE_STRIP`, `TRIANGLE_FAN`, `LINES`, `LINE_STRIP`, `POINTS` |
| `BlendMode` | `OFF`, `ALPHA`, `PREMULTIPLIED`, `ADDITIVE` |
| `DepthMode` | `OFF`, `TEST`, `TEST_AND_WRITE` |

Пятый аргумент включает отсечение задних граней; без него рисуются обе стороны каждого треугольника.

Думать стоит про `DepthMode`. `TEST_AND_WRITE` — то, что нужно плотной геометрии: она прячется и за блоками, и сама за собой. `TEST` прячет за миром, но не за твоими же отрисовками — так нужно всему полупрозрачному, что накладывается само на себя, и частицам в первую очередь: запись глубины порезала бы их на видимые квадраты. `OFF` рисует поверх всего.

## Матрицы

За тебя в шейдер не передаётся ничего. Матрицы кадра лежат в событии, и выставляешь их ты сам:

```kotlin
shader.setMat4("u_view", event.viewMatrix())
shader.setMat4("u_projection", event.projectionMatrix())
```

Обе — шестнадцать float'ов по столбцам, готовые для `mat4`. Минимальный вершинный шейдер:

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

Матрицу можно собрать и самому — например, модельную — и передать теми же шестнадцатью числами в том же порядке.

## Текстуры

`set(uniform, texture)` привязывает текстуру к сэмплеру с той фильтрацией, которая у неё уже есть, а у большинства игровых текстур это ближайший сосед. Проси то, что нужно:

```kotlin
val sprite = texture("minecraft:textures/particle/glow.png")

shader.set("u_texture", sprite, TextureFilter.LINEAR, TextureWrap.CLAMP)
```

`TextureFilter` — `NEAREST` или `LINEAR`, `TextureWrap` — `CLAMP` или `REPEAT`. Выбор живёт на отрисовке, а не на текстуре, так что линейная фильтрация здесь не изменит вид этой текстуры где-либо ещё в игре.

## Инстансинг

Для множества копий одной фигуры — частиц, маркеров, поля квадов — пиши одну запись на копию, а не на вершину. Отметь атрибуты через `perInstance()`:

```kotlin
val format = gpu.format(
    VertexAttribute.floats(3).perInstance(),   // центр
    VertexAttribute.floats(1).perInstance(),   // размер
    VertexAttribute.color().perInstance()
)

val pass = gpu.instancedRenderType(pipeline, mesh, 6)
```

Последний аргумент — сколько вершин у одной копии, шесть для квада из двух треугольников. Позиций вершинный шейдер не получает вообще: он строит их из `gl_VertexID` и данных инстанса. Биллборд, всегда повёрнутый к камере, получается сам собой, если делать это во view-пространстве, где оси экрана — просто x и y:

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

И `pass.drawInstanced()` вместо `pass.draw()`.

## Время, и почему `millis()` тебя укусит

Анимируй по счётчику тиков, а не по часам:

```kotlin
val time = (client.tick() + event.tickDelta()) / 20f
```

`client.millis()` — это настенное время, сегодня около 1.8e12. Поделив на тысячу и положив во `Float`, ты получаешь значения, соседние из которых отстоят уже на 128, так что `sin(time + phase)` вернёт одно и то же число для любого мыслимого `phase`. Анимации замирают, а кольцо частиц схлопывается в одну точку. Счётчик тиков стартует с нуля при запуске клиента и остаётся маленьким.

## Время жизни

Освободить всё это из скрипта нельзя, и не нужно. Выключение скрипта освобождает буферы, включение обратно — пересоздаёт их на следующем кадре. Перезагрузка или удаление скрипта отпускает всё, и консоль пишет, что именно ушло.

На видеокарте ничего не выделяется до первой отрисовки, так что загруженный, но выключенный скрипт не стоит ничего, сколько бы геометрии он ни объявлял.

## Лимиты

На скрипт даётся 64 ресурса — меши, пайплайны и render type'ы вместе, — а формат принимает не больше 16 атрибутов. В один меш влезает миллион вершин и четыре миллиона индексов. Выход за любой из лимитов бросает ошибку в скрипте, на той строке, которая запросила лишнее.
