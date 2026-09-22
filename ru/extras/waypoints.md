# Путевые точки

`client.waypoints()` ставит клиентские метки, привязанные к текущему домену сервера. С длительностью метка временная, без неё — постоянная и сохраняется в конфиг путевых точек.

```kotlin
val waypoints = client.waypoints()

waypoints.add("Base", player.position())            // постоянная
waypoints.add("Drop", player.position(), 20 * 60)   // 60 секунд

waypoints.remove("Drop")
waypoints.all().forEach { log.info(it.name()) }
```

## Добавить

| Метод | Тип | Описание |
|---|---|---|
| `waypoints.add(name, position, durationTicks)` | `Waypoint` | временная метка, исчезает через `durationTicks × 50` мс (бросает `ScriptException` на пустое имя или длительность меньше 1) |
| `waypoints.add(name, position)` | `Waypoint` | постоянная метка, сохраняется в конфиг путевых точек (бросает `ScriptException` на пустое имя) |
| `waypoints.remove(name)` | `void` | убирает все метки с таким именем (бросает `ScriptException` на пустое имя) |
| `waypoints.all()` | `List<Waypoint>` | снимок всех текущих меток, включая party-метки |

`position` — это [`Vec`](../game/math.md) в блоках; имена не уникальны. `client.waypoints()` бросает `ScriptStateException` после выгрузки скрипта.

## Метка

| Метод | Тип | Описание |
|---|---|---|
| `name()` | `String` | имя метки, как её зарегистрировали |
| `position()` | `Vec` | позиция в мире, в блоках |
| `permanent()` | `boolean` | true для сохранённой метки, false для временной |
| `remove()` | `void` | убирает все метки с этим именем |
