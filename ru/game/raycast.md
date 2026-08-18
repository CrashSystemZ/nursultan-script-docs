# Лучи и прицел

`raycast` — это `game.raycast()`. Любой метод требует клиентского потока, игрока и мира; расстояния — в блоках от начала луча.

```kotlin
on<ClientTickEvent> {
    val hit = raycast.crosshair(4.5)
    when {
        hit.isBlock() -> chat.print("block " + (hit as Hit.OnBlock).side())
        hit.isEntity() -> chat.print("entity " + (hit as Hit.OnEntity).entity().name())
        hit.isMiss() -> chat.print("miss")
    }
}
```

## Бросить луч

| Метод | Тип | Описание |
|---|---|---|
| `raycast.crosshair(maxDistanceBlocks)` | `Hit` | луч из прицела по блокам и всем задеваемым сущностям (бросает `ScriptException`, если `maxDistanceBlocks <= 0`) |
| `raycast.crosshair(maxDistanceBlocks, includeBlocks, entityFilter)` | `Hit` | то же с переключателем блоков и фильтром; null-фильтр пропускает всех (бросает `ScriptException`, если `maxDistanceBlocks <= 0`) |
| `raycast.entityAtCrosshair(maxDistanceBlocks)` | `Entity?` | сущность под прицелом, блоки перекрывают |
| `raycast.entityAtCrosshair(maxDistanceBlocks, filter)` | `Entity?` | то же с фильтром сущностей |
| `raycast.from(rotation, maxDistanceBlocks, includeBlocks, entityFilter)` | `Hit` | тот же луч из глаз, но вдоль произвольного поворота (бросает `ScriptException`, если `rotation` null или `maxDistanceBlocks <= 0`) |
| `raycast.blocks(from, to, shape, fluids)` | `Hit` | луч только по блокам между двумя точками; null-форма — `COLLIDER`, null-жидкости — `NONE` |
| `raycast.canSee(from, to)` | `boolean` | true, когда отрезок не перекрыт ни одним `COLLIDER`-блоком, жидкости игнорируются |
| `raycast.hitOn(entity, from, to)` | `Vec?` | точка на хитбоксе, расширенном запасом наводки, `from` если отрезок начинается внутри |

Зрителей и сущностей, по которым нельзя попасть, луч не возвращает никогда, с фильтром или без.
Вне клиентского потока любой метод бросает `ScriptThreadException`; без игрока или мира — `ScriptStateException`. `Vec` и `Rotation` — на странице [Векторы, коробки, углы](math.md).

## Попадание

**`Hit`**

| Метод | Тип | Описание |
|---|---|---|
| `hit.position()` | `Vec` | точка попадания в мировых координатах |
| `hit.distance()` | `double` | расстояние в блоках от начала луча до `position()` |
| `hit.isBlock()` | `boolean` | результат — `Hit.OnBlock` |
| `hit.isEntity()` | `boolean` | результат — `Hit.OnEntity` |
| `hit.isMiss()` | `boolean` | результат — `Hit.None` |

При промахе `blocks(...)` `position()` равна `to`; если трассировка из прицела не вернула ничего — это начало луча, а `distance()` равна 0.

**`Hit.OnBlock`, `Hit.OnEntity`**

| Метод | Тип | Описание |
|---|---|---|
| `Hit.OnBlock.blockX()` | `int` | координата x задетого блока |
| `Hit.OnBlock.blockY()` | `int` | координата y задетого блока |
| `Hit.OnBlock.blockZ()` | `int` | координата z задетого блока |
| `Hit.OnBlock.side()` | `Side` | задетая грань блока |
| `Hit.OnEntity.entity()` | `Entity` | сущность, в которую попал луч |

`Hit` запечатан над `OnBlock`, `OnEntity` и `None`; у `Hit.None` своих методов нет.

## Какая форма

**`RaycastShape`**

| Константа | Описание |
|---|---|
| `COLLIDER` | форма столкновений |
| `OUTLINE` | форма контура, та же что у выделения |
| `VISUAL` | видимая форма |

**`FluidHandling`**

| Константа | Описание |
|---|---|
| `NONE` | жидкости никогда не останавливают луч |
| `SOURCE_ONLY` | луч останавливают только источники |
| `ANY` | останавливает любая жидкость, включая текущую |

## Стороны

**`Side`**

| Константа | Описание |
|---|---|
| `DOWN` | грань -Y |
| `UP` | грань +Y |
| `NORTH` | грань -Z |
| `SOUTH` | грань +Z |
| `WEST` | грань -X |
| `EAST` | грань +X |

| Метод | Тип | Описание |
|---|---|---|
| `side.opposite()` | `Side` | грань на другом конце той же оси |

`Side` — это то же значение, которое принимают [`interaction.useBlock` / `placeBlock` / `startBreaking`](../actions/interaction.md).
