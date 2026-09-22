# Waypoints

`client.waypoints()` places client waypoints bound to the current server domain. A duration makes one temporary; no duration makes it permanent and saved into the waypoint config.

```kotlin
val waypoints = client.waypoints()

waypoints.add("Base", player.position())            // permanent
waypoints.add("Drop", player.position(), 20 * 60)   // 60 seconds

waypoints.remove("Drop")
waypoints.all().forEach { log.info(it.name()) }
```

## Adding one

| Method | Type | Description |
|---|---|---|
| `waypoints.add(name, position, durationTicks)` | `Waypoint` | temporary waypoint, expires after `durationTicks × 50` ms (throws `ScriptException` on a blank name or a duration below 1) |
| `waypoints.add(name, position)` | `Waypoint` | permanent waypoint, saved in the waypoint config (throws `ScriptException` on a blank name) |
| `waypoints.remove(name)` | `void` | removes every waypoint carrying that name (throws `ScriptException` on a blank name) |
| `waypoints.all()` | `List<Waypoint>` | snapshot of all current waypoints, party ones included |

`position` is a [`Vec`](../game/math.md) in blocks; names are not unique. `client.waypoints()` throws `ScriptStateException` after the script is unloaded.

## The marker

| Method | Type | Description |
|---|---|---|
| `name()` | `String` | waypoint name as registered |
| `position()` | `Vec` | world position in blocks |
| `permanent()` | `boolean` | true for a stored waypoint, false for a timed one |
| `remove()` | `void` | removes every waypoint carrying this name |
