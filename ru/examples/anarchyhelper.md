# Помощник на анархии

Следит за чатом и за здоровьем, ставит метки на смерти рядом, предупреждает о низком здоровье и умеет запоминать точки командой. Пример скрипта, который собирает вместе пакеты, конфиг, команду и уведомления.

```kotlin
name("AnarchyHelper")
description("Метки, предупреждения и заметки на анархии")

key(Key.H)

val warnHealth = slider("Предупредить при здоровье", 8f, 1f, 20f, 1f)
val markDeaths = checkBox("Отмечать смерти", true)
val deathMinutes = slider("Метка живёт, минут", 2, 1, 10)
    .visibleWhen { markDeaths.value() }
val notifyChat = checkBox("Ловить упоминания", true)
val watchWords = input("Слова через запятую", "тп,тпа,помоги")

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
            notify("здоровье " + health.toInt(), NotifyKind.WARN)
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
        notify("в чате: " + text.take(48), NotifyKind.ACCENT)
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
    chat.print("отметил место, где " + entity.name() + " лёг")
}

command("note") {
    usage("<add|go|list|del>")
    alias("n")

    sub("add") {
        usage("<имя>")
        runs {
            val name = arg(0)
            val position = player.position()
            notes.put(name, position.x().toInt().toString() + " "
                + position.y().toInt() + " " + position.z().toInt())
            notes.save()
            reply("записал " + name)
        }
    }

    sub("go") {
        usage("<имя>")
        completes(0) { notes.keys().toList() }
        runs {
            val value = notes.get(arg(0), "")
            if (value.isEmpty()) {
                replyError("нет заметки " + arg(0))
                return@runs
            }
            val parts = value.split(" ")
            client.waypoints().add(arg(0), Vec.of(
                parts[0].toDouble(), parts[1].toDouble(), parts[2].toDouble()
            ))
            reply("поставил метку на " + value)
        }
    }

    sub("del") {
        usage("<имя>")
        completes(0) { notes.keys().toList() }
        runs {
            notes.remove(arg(0))
            notes.save()
            reply("удалил " + arg(0))
        }
    }

    sub("list") {
        runs {
            if (notes.keys().isEmpty()) {
                reply("заметок нет")
                return@runs
            }
            reply("заметки: " + notes.keys().joinToString(", "))
        }
    }
}
```

## Что тут стоит заметить

**Пакеты читаются на сетевом потоке, уведомления показываются на клиентском.** Разбор текста можно делать сразу, а всё, что трогает игру, обёрнуто в `onClientThread`.

**Три разных пакета для одного и того же.** Сообщение в чате приезжает в разном виде в зависимости от того, как сервер его отправил — поэтому в `when` три ветки.

**Предупреждение с задержкой.** Без `sinceWarn` уведомление лезло бы каждый тик, пока здоровье низкое. `passedAndReset(40)` даёт не чаще раза в две секунды, и только когда здоровье уменьшилось.

**Конфиг вместо переменной.** Заметки нужны и после перезапуска, поэтому лежат в `config("notes")` и сохраняются сразу после изменения — команда вызывается редко, диск не пострадает.

**Подсказки берутся из тех же данных.** `completes(0) { notes.keys().toList() }` — и Tab предлагает ровно то, что записано.

**Настройка, которая прячется.** `deathMinutes` не имеет смысла, если метки выключены, поэтому у неё `visibleWhen`.
