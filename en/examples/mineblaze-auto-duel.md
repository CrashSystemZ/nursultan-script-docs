# MineBlaze auto duel

Answers `/next` when the server offers another round, and can throw an `ez` in first. The smallest useful shape a server-specific script has: read the chat out of the packet, act on the client thread.

```kotlin
name("MineBlaze Auto Duel")
description("Answers /next when a duel offers another round, and can say ez first")

val autoEz = checkBox("Auto EZ", false)

val PREFIX = "Дуэли"
val TRIGGER = "Хотите продолжить?"
val COOLDOWN_MS = 5000L

var lastSent = 0L

onEnable { lastSent = 0L }

on<PacketReceiveEvent> { e ->
    val text = when (val packet = e.packet()) {
        is S2CGameMessagePacket -> if (packet.overlay()) null else packet.content()
        is S2CProfilelessChatMessagePacket -> packet.message()
        is S2CChatMessagePacket -> packet.unsignedContent()
        else -> null
    }
    if (text == null || !text.contains(PREFIX) || !text.contains(TRIGGER)) return@on
    onClientThread {
        val now = client.millis()
        if (now - lastSent < COOLDOWN_MS) return@onClientThread
        lastSent = now
        if (autoEz.value()) {
            chat.sendToServer("ez")
        }
        chat.sendToServer("/next")
        log.info("offer to continue -> /next")
    }
}
```

## Things worth noticing

**The constants are the server's own words.** `PREFIX` and `TRIGGER` are matched against what MineBlaze prints, so they are written exactly as the server writes them — including the language. Point this at another server and those two lines are what you change.

**Three different packets for one chat message.** The same text arrives in a different shape depending on how the server sent it, which is why the `when` has three branches. `overlay()` is the action bar, not the chat, so it is skipped.

**Parsing on the network thread, sending on the client one.** The `when` runs where the packet arrives. Everything after it — reading the setting, sending chat — is inside `onClientThread`.

**A cooldown, because the offer can arrive twice.** Servers repeat that line, and two `/next` in a row would queue you into a round you did not want. `COOLDOWN_MS` makes the second one a no-op. Real time is right here rather than ticks, because the interval is the server's, not the game's.

**`chat.sendToServer` is exactly what typing it would do.** `ez` goes out as a chat message and `/next` as a command, for the same reason they would from the chat box.
