# Anarchy helper

Watches chat and your health, marks nearby deaths, warns on low health, and remembers spots via a command. An example of a script that ties packets, a config, a command and notifications together.

```kotlin
name("AnarchyHelper")
description("Marks, warnings and notes for anarchy")

key(Key.H)

val warnHealth = slider("Warn at health", 8f, 1f, 20f, 1f)
val markDeaths = checkBox("Mark deaths", true)
val deathMinutes = slider("Mark lives, minutes", 2, 1, 10)
    .visibleWhen { markDeaths.value() }
val notifyChat = checkBox("Catch mentions", true)
val watchWords = input("Words, comma separated", "tp,tpa,help")

val notes = config("notes")
val sinceWarn = timer()

var lastHealth = 20f

onEnable {
    lastHealth = 20f
    sinceWarn.reset()
}

fun words(): List<String> =
    watchWords.value()
        .split(",")
        .map { it.trim().lowercase() }
        .filter { it.isNotEmpty() }

on<ClientTickEvent> {
    whenInGame {
        val health = player.health()
        if (health < warnHealth.value() && health < lastHealth && sinceWarn.passedAndReset(40)) {
            notify("health " + health.toInt(), NotifyKind.WARN)
        }
        lastHealth = health
    }
}

on<PacketReceiveEvent> { e ->
    if (!notifyChat.value()) return@on
    val text = when (val packet = e.packet()) {
        is S2CGameMessagePacket -> if (packet.overlay()) null else packet.content()
        is S2CProfilelessChatMessagePacket -> packet.message()
        is S2CChatMessagePacket -> packet.unsignedContent()
        else -> null
    } ?: return@on

    val lower = text.lowercase()
    if (words().none { lower.contains(it) }) return@on

    onClientThread {
        notify("chat: " + text.take(48), NotifyKind.ACCENT)
    }
}

on<EntityRemoveEvent> { e ->
    if (!markDeaths.value()) return@on
    val entity = e.entity()
    if (!entity.isPlayer() || entity.isSelf()) return@on
    val living = entity.asLiving() ?: return@on
    if (living.health() > 0f) return@on
    if (entity.distanceTo(player.position()) > 48.0) return@on

    client.waypoints().add(entity.name(), entity.position(), deathMinutes.intValue() * 20 * 60)
    chat.print("marked where " + entity.name() + " went down")
}

command("note") {
    usage("<add|go|list|del>")
    alias("n")

    sub("add") {
        usage("<name>")
        runs {
            val name = arg(0)
            val position = player.position()
            notes.put(name, position.x().toInt().toString() + " "
                + position.y().toInt() + " " + position.z().toInt())
            notes.save()
            reply("saved " + name)
        }
    }

    sub("go") {
        usage("<name>")
        completes(0) { notes.keys().toList() }
        runs {
            val value = notes.get(arg(0), "")
            if (value.isEmpty()) {
                replyError("no note called " + arg(0))
                return@runs
            }
            val parts = value.split(" ")
            client.waypoints().add(arg(0), Vec.of(
                parts[0].toDouble(), parts[1].toDouble(), parts[2].toDouble()
            ))
            reply("marked " + value)
        }
    }

    sub("del") {
        usage("<name>")
        completes(0) { notes.keys().toList() }
        runs {
            notes.remove(arg(0))
            notes.save()
            reply("removed " + arg(0))
        }
    }

    sub("list") {
        runs {
            if (notes.keys().isEmpty()) {
                reply("no notes")
                return@runs
            }
            reply("notes: " + notes.keys().joinToString(", "))
        }
    }
}
```

## Things worth noticing

**Packets are read on the network thread, notifications shown on the client one.** Parsing the text can happen right away; anything touching the game is wrapped in `onClientThread`.

**Three different packets for the same thing.** A chat message arrives in different shapes depending on how the server sent it, which is why the `when` has three branches.

**A warning with a cooldown.** Without `sinceWarn` the notification would fire every tick while health is low. `passedAndReset(40)` limits it to once every two seconds, and only when health actually dropped.

**A config instead of a variable.** Notes have to survive a restart, so they live in `config("notes")` and are saved right after a change — the command runs rarely, so the disk is fine.

**Completions come from the same data.** `completes(0) { notes.keys().toList() }` means Tab offers exactly what is written down.

**A setting that hides itself.** `deathMinutes` means nothing when marks are off, so it has a `visibleWhen`.
