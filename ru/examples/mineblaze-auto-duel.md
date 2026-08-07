# Автодуэль на MineBlaze

Отвечает `/next`, когда сервер предлагает следующий раунд, и по желанию пишет перед этим `ez`. Самая маленькая полезная форма скрипта под конкретный сервер: прочитать чат из пакета и сделать дело на клиентском потоке.

```kotlin
name("MineBlaze Auto Duel")
description("Отвечает /next на предложение продолжить дуэль, по желанию пишет ez")

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
        log.info("предложение продолжить -> /next")
    }
}
```

## Что тут стоит заметить

**Константы — это слова самого сервера.** `PREFIX` и `TRIGGER` сравниваются с тем, что печатает MineBlaze, поэтому написаны ровно так, как их пишет сервер. Перенесёшь скрипт на другой — менять надо именно эти две строки.

**Три разных пакета для одного сообщения в чате.** Один и тот же текст приезжает в разном виде в зависимости от того, как сервер его отправил, — поэтому в `when` три ветки. `overlay()` — это строка над хотбаром, а не чат, её пропускаем.

**Разбор на сетевом потоке, отправка на клиентском.** `when` выполняется там, где пришёл пакет. Всё, что после, — чтение настройки и отправка в чат — уже внутри `onClientThread`.

**Кулдаун, потому что предложение приходит дважды.** Серверы повторяют такую строку, и два `/next` подряд закинут тебя в раунд, которого ты не хотел. `COOLDOWN_MS` делает второй пустышкой. Реальное время здесь уместнее тиков: интервал серверный, а не игровой.

**`chat.sendToServer` — ровно то же, что набрать это руками.** `ez` уходит обычным сообщением, `/next` — командой, по той же причине, по которой так вышло бы из чата.
