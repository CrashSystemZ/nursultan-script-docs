# Пакеты

Пакет — это сообщение между клиентом и сервером. Через пакеты видно то, что не видно иначе: точный момент отбрасывания, системные сообщения, действия других игроков.

## Два потока

Пакеты приходят **не на клиентском потоке**. Читать поля пакета там можно и нужно, а вот трогать мир, игрока или инвентарь — нельзя. Для этого перепрыгни:

```kotlin
on<PacketReceiveEvent> { e ->
    val packet = e.packet()
    if (packet !is S2CEntityVelocityPacket) return@on
    if (packet.entityId() != player.id()) return@on

    onClientThread {
        control.jump()
    }
}
```

Забудешь про `onClientThread` — получишь редкие странные баги, которые тяжело поймать.

## Что приходит

```kotlin
on<PacketReceiveEvent> { e -> e.packet() }   // от сервера, S2CPacket
on<PacketSendEvent> { e -> e.packet() }      // от клиента, C2SPacket
```

Пакеты — это Kotlin-записи, поэтому разбирать их удобнее всего через `when`:

```kotlin
on<PacketReceiveEvent> { e ->
    val text = when (val packet = e.packet()) {
        is S2CGameMessagePacket -> if (packet.overlay()) null else packet.content()
        is S2CProfilelessChatMessagePacket -> packet.message()
        is S2CChatMessagePacket -> packet.unsignedContent()
        else -> null
    }
    if (text != null && text.contains("дуэль")) {
        onClientThread { chat.sendToServer("/next") }
    }
}
```

## Отмена

И входящие, и исходящие пакеты можно отменить:

```kotlin
on<PacketSendEvent> { e ->
    if (e.packet() is C2SClientTickEndPacket) {
        e.cancel()
    }
}
```

Отменяй с осторожностью: сервер ждёт от клиента определённого поведения, и слишком творческий подход заканчивается киком.

## Отправить свой

```kotlin
game.packets().send(C2SChatMessagePacket("привет", 0L, 0L))
game.packets().send(C2SUpdateBeaconPacket("minecraft:speed", "minecraft:haste"))
game.packets().send(C2SSpectatorTeleportPacket(target.uuid()))
game.packets().sendSequenced(C2SPlayerActionPacket(...))
```

`send` возвращает `false`, если отправить не получилось — например, нет подключения.

`sendSequenced` нужен для пакетов, которые сервер нумерует: взаимодействия с блоками и предметами. Если сомневаешься — обычный `send`.

## Придержать пакет и отправить позже

Отменить пакет и отправить тот же объект потом — так задерживают собственный поток: клиент играет как обычно, а сервер узнаёт об этом чуть позже:

```kotlin
val held = mutableListOf<C2SPacket>()
var releasing = false

on<PacketSendEvent> { e ->
    if (releasing) return@on
    held.add(e.packet())
    e.cancel()
}

fun release() {
    releasing = true
    for (packet in held) {
        game.packets().send(packet)
    }
    held.clear()
    releasing = false
}
```

Флаг `releasing` тут не для красоты: то, что ты отправляешь, снова приходит в `PacketSendEvent`, и без него пакеты будут вставать в очередь бесконечно.

Два момента. Нумерованные пакеты хранят свой номер — `send` его переиспользует, поэтому переотправленное ломание блока не разъезжается с предсказанием клиента. А пакеты, называющие сущность, в первую очередь `C2SPlayerInteractEntityPacket`, собираются обратно поиском id в мире: если цель умерла, пока пакет лежал в списке, `send` вернёт `false` и пакет пропадёт.

Переотправить можно не всё. Подписанный чат, клики по инвентарю и плагин-сообщения несут то, чего в скриптовой записи нет (подпись, стаки, сырые байты), — `send` их не примет и напишет об этом в консоль скриптов. Такие пропускай, а не придерживай.

## Как называются пакеты

Имена собраны из ванильных: приставка `C2S` — от клиента к серверу, `S2C` — от сервера к клиенту.

Чаще всего нужны:

| Пакет | Про что |
|---|---|
| `S2CEntityVelocityPacket` | кого-то оттолкнуло |
| `S2CGameMessagePacket`, `S2CChatMessagePacket`, `S2CProfilelessChatMessagePacket` | сообщения в чате |
| `S2CEntityDamagePacket` | кому-то нанесли урон |
| `S2CEntityStatusEffectPacket` | кому-то повесили эффект — какой именно, скажет `effect()` |
| `S2CEntityEquipmentUpdatePacket` | кто-то сменил то, что держит или носит |
| `S2CEntityAttributesPacket` | у кого-то изменились макс. здоровье, скорость или урон |
| `S2CInventoryPacket`, `S2CScreenHandlerSlotUpdatePacket` | обновился инвентарь |
| `S2CExplosionPacket` | взрыв |
| `S2CBlockEntityUpdatePacket` | обновилась блок-сущность — в `nbt()` лежат новые данные |
| `S2CSetTradeOffersPacket` | сделки торговца |
| `S2CEntityPositionPacket`, `S2CEntityMoveRelativePacket` | сущность подвинулась |
| `C2SMoveFullPacket`, `C2SMoveLookPacket`, `C2SMoveOnGroundPacket` | наше движение |
| `C2SChatMessagePacket`, `C2SCommandExecutionPacket` | что мы пишем |
| `C2SPlayerInteractEntityPacket`, `C2SPlayerActionPacket` | наши действия |

Полный список — в автодополнении IDE: набери `S2C` или `C2S` и смотри, что предлагается. Все они уже импортированы, писать `import` не нужно.

## Пакеты со списком внутри

Некоторые пакеты несут не только числа — список предметов, атрибутов, сделок. Такое приходит вложенной записью и читается обычным циклом:

```kotlin
on<PacketReceiveEvent> { e ->
    val packet = e.packet()
    if (packet !is S2CEntityEquipmentUpdatePacket) return@on

    for (slot in packet.equipment()) {
        log.info("${slot.slot()} = ${slot.item()} x${slot.count()}")
    }
}
```

В `slot()` лежат ванильные имена — `mainhand`, `offhand`, `head`, `chest`, `legs`, `feet`, `body`, `saddle`. Так смена оружия видна в момент, когда она произошла, а не когда по тебе уже замахнулись.

Та же форма в остальных:

| Пакет | Список | Одна запись |
|---|---|---|
| `S2CEntityEquipmentUpdatePacket` | `equipment()` | `slot()`, `item()`, `count()` |
| `S2CInventoryPacket` | `contents()` | `item()`, `count()` |
| `S2CEntityAttributesPacket` | `attributes()` | `attribute()`, `base()`, `modifiers()` |
| `S2CSetTradeOffersPacket` | `offers()` | что торговец берёт и что даёт, плюс `uses()`, `maxUses()`, `disabled()` |
| `S2CStatisticsPacket` | `stats()` | `stat()`, `value()` |
| `S2CEntityTrackerUpdatePacket` | `values()` | `id()`, `value()` |

Две оговорки. Предмет здесь — это идентификатор и количество, а не полноценный [предмет](../game/inventory.md): пакет — то, что пришло по проводу, зачарований и лора в нём нет. Нужны они — смотри слот или сущность в мире. И `S2CEntityTrackerUpdatePacket` отдаёт то, что игра отслеживала — позу, здоровье, флаги — приведённое к тексту, с числовым `id()`, который сам по себе ничего не значит. Годится заметить, что что-то изменилось, а не разобрать, что именно.

## Привязка к версии Minecraft

Пакеты — единственная часть API, привязанная к версии игры. При обновлении Minecraft поля и имена пакетов могут поменяться. Если твой скрипт их использует, распространяй его вместе с подходящей версией клиента и обнови джарку пакетов в `.sdk/`, когда обновляешься сам.

Скрипты, которые пакеты не трогают, обновление Minecraft переживают без правок.
