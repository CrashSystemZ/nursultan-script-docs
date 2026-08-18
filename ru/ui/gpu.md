# Своя геометрия

`gpu` — это `client.gpu()`: свои форматы вершин, меши, пайплайны и вызовы отрисовки. Координаты вершин задаются относительно камеры, буферы освобождаются, пока скрипт выключен, и всё на этой странице — API 2.

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

## Фабрика

| Метод | Тип | Описание |
|---|---|---|
| `gpu.format(attributes...)` | `VertexFormat` | чередующийся формат вершины (бросает `IllegalArgumentException` меньше 1 и больше 16 атрибутов) |
| `gpu.mesh(format)` | `Mesh` | меш без индексов, стартовый буфер 1024 байта (бросает `IllegalArgumentException` на чужом формате) |
| `gpu.indexedMesh(format)` | `Mesh` | меш с индексным буфером, по 1024 байта |
| `gpu.pipeline(shader, drawMode, blend, depth)` | `Pipeline` | отсечение граней выключено |
| `gpu.pipeline(shader, drawMode, blend, depth, cull)` | `Pipeline` | null заменяется на `TRIANGLES`/`ALPHA`/`OFF` (бросает `IllegalArgumentException` на чужом шейдере) |
| `gpu.renderType(pipeline, mesh)` | `RenderType` | связка без инстансинга |
| `gpu.instancedRenderType(pipeline, mesh, verticesPerInstance)` | `RenderType` | `verticesPerInstance` зажимается от 0 (бросает `IllegalArgumentException` на чужом пайплайне или меше) |

Каждый метод бросает `IllegalStateException` после закрытия сессии скрипта. Меши, пайплайны и render type'ы — учитываемые ресурсы, форматы — нет.

## Формат вершины

| Метод | Тип | Описание |
|---|---|---|
| `VertexAttribute(count, type, normalized, instanced)` | `VertexAttribute` | канонический конструктор |
| `attribute.count()` | `int` | количество компонент |
| `attribute.type()` | `AttributeType` | тип компоненты |
| `attribute.normalized()` | `boolean` | флаг нормализации, игнорируется — нормализуется только `UNSIGNED_BYTE` |
| `attribute.instanced()` | `boolean` | true, когда атрибут шагает по инстансам |
| `VertexAttribute.floats(count)` | `VertexAttribute` | статический, `count` float'ов, на вершину |
| `VertexAttribute.ints(count)` | `VertexAttribute` | статический, `count` целых, на вершину |
| `VertexAttribute.color()` | `VertexAttribute` | статический, 4 нормализованных байта, на вершину |
| `attribute.perInstance()` | `VertexAttribute` | копия с `instanced = true` |

| Константа | Описание |
|---|---|
| `AttributeType.FLOAT` | 32-битные float-компоненты |
| `AttributeType.INT` | 32-битные знаковые целые компоненты |
| `AttributeType.UNSIGNED_BYTE` | 4 нормализованных байта независимо от указанного count |

| Метод | Тип | Описание |
|---|---|---|
| `format.stride()` | `int` | байт на вершину |
| `format.attributeCount()` | `int` | сколько атрибутов в формате |

Атрибуты ложатся на входы шейдера `layout(location = N)` в порядке объявления; совпадение записи с форматом никто не проверяет.

## Запись вершин

| Метод | Тип | Описание |
|---|---|---|
| `mesh.format()` | `VertexFormat` | формат, с которым создан меш |
| `mesh.verts()` | `VertexWriter` | писатель вершин (бросает `IllegalStateException`, когда меш закрыт) |
| `mesh.idx()` | `IndexWriter?` | писатель индексов, null у меша без индексов |
| `mesh.indexed()` | `boolean` | true, если создан через `gpu.indexedMesh` |
| `mesh.clear()` | `void` | сбрасывает оба писателя, ничего не делает у неиспользованного меша |

GL-буфер создаётся при первом использовании, и первая запись должна идти с рендер-потока. `draw()` загружает оба писателя и очищает их, так что каждый кадр пишет геометрию заново.

| Метод | Тип | Описание |
|---|---|---|
| `verts.putFloat(value)` | `VertexWriter` | пишет 4 байта |
| `verts.putInt(value)` | `VertexWriter` | пишет 4 байта |
| `verts.putVec2(x, y)` | `VertexWriter` | пишет два float'а, то же самое, что `putUv` |
| `verts.putVec3(x, y, z)` | `VertexWriter` | пишет три float'а |
| `verts.putVec3(position)` | `VertexWriter` | то же из [`Vec`](../game/math.md), компоненты приводятся к float |
| `verts.putUv(u, v)` | `VertexWriter` | пишет два float'а |
| `verts.putColor(argb)` | `VertexWriter` | пишет ARGB-число; в GLSL приходит как BGRA |
| `verts.next()` | `int` | закрывает вершину, возвращает её индекс с нуля (бросает `IllegalStateException` на 1048576 вершинах) |
| `verts.vertexCount()` | `int` | сколько вершин закрыто, 0 у закрытого меша |

## Индексы

| Метод | Тип | Описание |
|---|---|---|
| `idx.put(index)` | `IndexWriter` | добавляет один индекс вершины (бросает `IllegalStateException` на 4194304 индексах) |
| `idx.putTriangle(a, b, c)` | `IndexWriter` | добавляет три индекса |
| `idx.putQuad(a, b, c, d)` | `IndexWriter` | добавляет треугольники `(a, b, c)` и `(a, c, d)` |
| `idx.indexCount()` | `int` | сколько индексов записано, 0 у закрытого меша |

## Пайплайн

| Метод | Тип | Описание |
|---|---|---|
| `pipeline.shader()` | `Shader` | шейдер, которым рисует пайплайн — [Шейдеры](shaders.md) |
| `pipeline.drawMode()` | `DrawMode` | тип примитива |

| Константа | Описание |
|---|---|
| `DrawMode.TRIANGLES` | `GL_TRIANGLES`, а также замена для null |
| `DrawMode.TRIANGLE_STRIP` | `GL_TRIANGLE_STRIP` |
| `DrawMode.TRIANGLE_FAN` | `GL_TRIANGLE_FAN` |
| `DrawMode.LINES` | `GL_LINES` |
| `DrawMode.LINE_STRIP` | `GL_LINE_STRIP` |
| `DrawMode.POINTS` | `GL_POINTS` |

| Константа | Описание |
|---|---|
| `BlendMode.OFF` | смешивание выключено |
| `BlendMode.ALPHA` | `SRC_ALPHA, ONE_MINUS_SRC_ALPHA`, замена для null |
| `BlendMode.PREMULTIPLIED` | `ONE, ONE_MINUS_SRC_ALPHA` по цвету и альфе |
| `BlendMode.ADDITIVE` | `SRC_ALPHA, ONE` по цвету и альфе |

| Константа | Описание |
|---|---|
| `DepthMode.OFF` | тест глубины выключен, запись выключена, замена для null |
| `DepthMode.TEST` | тест глубины включён, запись выключена |
| `DepthMode.TEST_AND_WRITE` | тест глубины включён, запись включена |

## Отрисовка

| Метод | Тип | Описание |
|---|---|---|
| `pass.pipeline()` | `Pipeline` | используемый пайплайн |
| `pass.mesh()` | `Mesh` | используемый меш |
| `pass.draw()` | `void` | биндит фреймбуфер клиента и делает один вызов отрисовки (бросает `IllegalStateException`, когда render type или меш закрыт) |
| `pass.drawInstanced()` | `void` | рисует инстансы по `verticesPerInstance` вершин, только для `instancedRenderType` |

Встроенных униформ здесь нет, в отличие от `Render.shader`: `u_view`, `u_projection` и всё остальное выставляешь сам до вызова. Оба метода ничего не рисуют, пока меш пуст или шейдер не собрался.

## Лимиты

| Лимит | Значение |
|---|---|
| живых gpu-ресурсов на скрипт | 64 меша, пайплайна и render type'а вместе |
| атрибутов на формат | 16 |
| вершин в меше | 1 048 576 |
| индексов в меше | 4 194 304 |
| встроенный шрифт или PNG | 8 МиБ |
