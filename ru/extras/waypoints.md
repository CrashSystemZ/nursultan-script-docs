# Путевые точки

Скрипт может ставить те же метки в мире, что и сам клиент: с подписью и расстоянием.

```kotlin
waypoints = client.waypoints()

waypoints.add("База", player.position())            // насовсем
waypoints.add("Дроп", position, 20 * 60)            // на минуту
waypoints.remove("Дроп")
waypoints.all()
```

Третий аргумент — сколько тиков метка проживёт. Без него она остаётся, пока её не уберут.

## Сама метка

```kotlin
val point = client.waypoints().all().first()

point.name()
point.position()
point.permanent()      // без срока
point.remove()
```

## Зачем это нужно

Отметить, где кто-то умер:

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

Отметить сундук, до которого не дошёл:

```kotlin
command("mark") {
    runs {
        val hit = raycast.crosshair(player.blockReachBlocks())
        if (hit !is Hit.OnBlock) {
            replyError("нет блока под прицелом")
            return@runs
        }
        client.waypoints().add(rest().ifEmpty { "метка" }, hit.position())
        reply("отмечено")
    }
}
```

## Что учесть

* Имена не уникальны, но `remove(name)` уберёт первую подходящую — так что лучше давать разные.
* Метки со сроком исчезают сами, чистить их не надо.
* Метки скрипта живут, пока живёт клиент; при перезагрузке скрипта они не восстанавливаются автоматически. Нужно постоянство — храни их в [конфиге](../settings/storage.md) и расставляй заново на `onEnable`.
