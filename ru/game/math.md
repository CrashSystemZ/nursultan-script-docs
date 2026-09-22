# Векторы, коробки, углы

`Vec`, `Box` и `Rotation` — неизменяемые значения, которые встречаются по всему API: позиции и скорости в блоках, ограничивающие коробки в мировых координатах, yaw и pitch в градусах. Это record'ы — любая операция возвращает новый объект, а `equals`/`hashCode` сравнивают компоненты.

```kotlin
on<ClientTickEvent> {
    val eyes = player.eyePosition()
    val ahead = player.rotation().direction().multiply(3.0)   // 3 блока вперёд
    val area = Box.around(eyes, 0.5).offset(ahead)
    chat.print("впереди: " + world.entitiesIn(area).size)
}
```

## Vec

| Метод | Тип | Описание |
|---|---|---|
| `Vec(x, y, z)` | `Vec` | канонический конструктор, компоненты в блоках, без нормализации |
| `Vec.of(x, y, z)` | `Vec` | статическая фабрика, то же самое, что конструктор |
| `Vec.ZERO` | `Vec` | константа (0, 0, 0) |
| `x()` | `double` | компонента x в блоках |
| `y()` | `double` | компонента y в блоках |
| `z()` | `double` | компонента z в блоках |
| `add(other)` | `Vec` | покомпонентная сумма |
| `add(x, y, z)` | `Vec` | покомпонентная сумма с литералами |
| `subtract(other)` | `Vec` | покомпонентная разность |
| `multiply(factor)` | `Vec` | равномерное умножение на число |
| `normalize()` | `Vec` | единичный вектор, `ZERO` при нулевой длине |
| `dot(other)` | `double` | скалярное произведение |
| `length()` | `double` | евклидова длина в блоках |
| `squaredLength()` | `double` | квадрат длины, без sqrt |
| `distanceTo(other)` | `double` | евклидово расстояние в блоках |
| `squaredDistanceTo(other)` | `double` | квадрат расстояния, без sqrt |

Позиции, точки глаз и скорости приходят как `Vec` — смотри [Свой игрок](player.md) и [Сущности и фильтры](entities.md).

## Box

| Метод | Тип | Описание |
|---|---|---|
| `Box(minX, minY, minZ, maxX, maxY, maxZ)` | `Box` | канонический конструктор, меняет компоненты местами, чтобы min ≤ max по каждой оси |
| `Box.around(center, radiusBlocks)` | `Box` | куб с полуразмером radiusBlocks вокруг точки |
| `minX()` | `double` | нижняя граница по x |
| `minY()` | `double` | нижняя граница по y |
| `minZ()` | `double` | нижняя граница по z |
| `maxX()` | `double` | верхняя граница по x |
| `maxY()` | `double` | верхняя граница по y |
| `maxZ()` | `double` | верхняя граница по z |
| `center()` | `Vec` | середина коробки |
| `sizeX()` | `double` | размер по x в блоках |
| `sizeY()` | `double` | размер по y в блоках |
| `sizeZ()` | `double` | размер по z в блоках |
| `offset(offset)` | `Box` | сдвинутая копия |
| `offset(x, y, z)` | `Box` | сдвинутая копия с литералами |
| `expand(blocks)` | `Box` | раздута на blocks со всех шести сторон |
| `expand(x, y, z)` | `Box` | раздута по осям, с обеих сторон |
| `contract(blocks)` | `Box` | сжата на blocks со всех шести сторон |
| `contains(point)` | `boolean` | точка внутри, границы включительно |
| `intersects(other)` | `boolean` | объёмы пересекаются, касание граней не считается |
| `raycast(from, to)` | `Vec?` | точка входа на отрезке, null при промахе, `from` если начало внутри |

`entity.box()` и `block.box()` возвращают её, а `world.entitiesIn`, `collisionsIn` и `isFree` её принимают — смотри [Мир и блоки](world.md).

## Rotation

| Метод | Тип | Описание |
|---|---|---|
| `Rotation(yawDegrees, pitchDegrees)` | `Rotation` | канонический конструктор, зажимает pitch в -90..90 градусов |
| `Rotation.of(yawDegrees, pitchDegrees)` | `Rotation` | статическая фабрика, то же самое, что конструктор |
| `yawDegrees()` | `float` | yaw в градусах, без нормализации |
| `pitchDegrees()` | `float` | pitch в градусах, -90..90 |
| `yawDeltaTo(other)` | `float` | кратчайшая знаковая разница yaw, -180..180 градусов |
| `pitchDeltaTo(other)` | `float` | знаковая разница pitch в градусах |
| `angleTo(other)` | `float` | гипотенуза разниц yaw и pitch, в градусах |
| `direction()` | `Vec` | единичный вектор направления взгляда |

Чтение, построение и применение поворотов — на странице [Повороты](../actions/rotations.md); `raycast.from(rotation, ...)` — на странице [Лучи и прицел](raycast.md).
