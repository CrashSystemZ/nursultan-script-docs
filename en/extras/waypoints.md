# Waypoints

A script can drop the same world markers the client uses, with a label and a distance.

```kotlin
waypoints = client.waypoints()

waypoints.add("Base", player.position())            // forever
waypoints.add("Drop", position, 20 * 60)            // for a minute
waypoints.remove("Drop")
waypoints.all()
```

The third argument is how many ticks the marker lives. Without it, it stays until removed.

## The marker itself

```kotlin
val point = client.waypoints().all().first()

point.name()
point.position()
point.permanent()      // no expiry
point.remove()
```

## What it is good for

Marking where someone died:

```kotlin
on<PacketReceiveEvent> { e ->
    val packet = e.packet()
    if (packet !is S2CEntityDamagePacket) return@on
    onClientThread {
        val victim = world.entityById(packet.entityId()) ?: return@onClientThread
        val living = victim.asLiving() ?: return@onClientThread
        if (living.health() > 0f) return@onClientThread
        client.waypoints().add(victim.name(), victim.position(), 20 * 120)
    }
}
```

Marking a chest you did not get to:

```kotlin
command("mark") {
    runs {
        val hit = raycast.crosshair(player.blockReachBlocks())
        if (hit !is Hit.OnBlock) {
            replyError("no block under the crosshair")
            return@runs
        }
        client.waypoints().add(rest().ifEmpty { "mark" }, hit.position())
        reply("marked")
    }
}
```

## Things to keep in mind

* Names are not unique, but `remove(name)` drops the first match — so give them different names.
* Timed markers expire on their own, nothing to clean up.
* A script's markers live as long as the client does; reloading the script does not bring them back. If you want them to persist, keep them in a [config](../settings/storage.md) and place them again in `onEnable`.
