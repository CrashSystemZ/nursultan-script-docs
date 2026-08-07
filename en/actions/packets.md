# Packets

A packet is a message between the client and the server. Packets show you things nothing else does: the exact moment of a knockback, system messages, what other players did.

## Two threads

Packets do **not** arrive on the client thread. Reading the packet's fields there is fine and expected; touching the world, the player or the inventory is not. Hop over first:

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

Forget `onClientThread` and you get rare, strange bugs that are painful to catch.

## What arrives

```kotlin
on<PacketReceiveEvent> { e -> e.packet() }   // from the server, S2CPacket
on<PacketSendEvent> { e -> e.packet() }      // from the client, C2SPacket
```

Packets are Kotlin records, so a `when` is the nicest way to take them apart:

```kotlin
on<PacketReceiveEvent> { e ->
    val text = when (val packet = e.packet()) {
        is S2CGameMessagePacket -> if (packet.overlay()) null else packet.content()
        is S2CProfilelessChatMessagePacket -> packet.message()
        is S2CChatMessagePacket -> packet.unsignedContent()
        else -> null
    }
    if (text != null && text.contains("duel")) {
        onClientThread { chat.sendToServer("/next") }
    }
}
```

## Cancelling

Both incoming and outgoing packets can be cancelled:

```kotlin
on<PacketSendEvent> { e ->
    if (e.packet() is C2SClientTickEndPacket) {
        e.cancel()
    }
}
```

Cancel carefully: the server expects certain behaviour from a client, and getting too creative ends in a kick.

## Sending your own

```kotlin
game.packets().send(C2SChatMessagePacket("hi", 0L, 0L))
game.packets().send(C2SUpdateBeaconPacket("minecraft:speed", "minecraft:haste"))
game.packets().send(C2SSpectatorTeleportPacket(target.uuid()))
game.packets().sendSequenced(C2SPlayerActionPacket(...))
```

`send` returns `false` when it could not go out — no connection, for instance.

`sendSequenced` is for packets the server numbers: block and item interactions. When in doubt, plain `send`.

## Holding a packet back and sending it later

Cancelling a packet and sending the same object later is how you delay your own stream — the client keeps playing, the server hears about it a moment afterwards:

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

The `releasing` flag matters: what you send comes back through `PacketSendEvent`, and without the guard you would queue it forever.

Two things to keep in mind. Sequenced packets carry the number they were sent with — `send` reuses it, so a replayed block break stays in step with the client's own prediction. And packets that name an entity, `C2SPlayerInteractEntityPacket` above all, are rebuilt by looking the id up in the world: if the target died while the packet sat in your list, `send` returns `false` and the packet is gone.

Not everything can be resent. Signed chat, inventory clicks and plugin messages carry data the script record does not (a signature, item stacks, raw bytes), so `send` refuses them and says so in the script console. Let those through instead of holding them.

## How packets are named

Names follow the vanilla ones: the `C2S` prefix is client to server, `S2C` is server to client.

The ones you will want most:

| Packet | What it is about |
|---|---|
| `S2CEntityVelocityPacket` | someone got knocked back |
| `S2CGameMessagePacket`, `S2CChatMessagePacket`, `S2CProfilelessChatMessagePacket` | chat messages |
| `S2CEntityDamagePacket` | someone took damage |
| `S2CEntityStatusEffectPacket` | someone got an effect — `effect()` says which |
| `S2CEntityEquipmentUpdatePacket` | someone changed what they hold or wear |
| `S2CEntityAttributesPacket` | someone's max health, speed or damage changed |
| `S2CInventoryPacket`, `S2CScreenHandlerSlotUpdatePacket` | the inventory changed |
| `S2CExplosionPacket` | an explosion |
| `S2CBlockEntityUpdatePacket` | a block entity changed — its `nbt()` is the new data |
| `S2CSetTradeOffersPacket` | a villager's trades |
| `S2CEntityPositionPacket`, `S2CEntityMoveRelativePacket` | an entity moved |
| `C2SMoveFullPacket`, `C2SMoveLookPacket`, `C2SMoveOnGroundPacket` | our movement |
| `C2SChatMessagePacket`, `C2SCommandExecutionPacket` | what we type |
| `C2SPlayerInteractEntityPacket`, `C2SPlayerActionPacket` | our actions |

The full list is in your IDE's autocompletion: type `S2C` or `C2S` and look at what comes up. They are all pre-imported, so you never write an `import`.

## Packets that carry a list

Some packets bring more than plain numbers — a list of items, of attributes, of trades. Those arrive as a nested record you read with an ordinary loop:

```kotlin
on<PacketReceiveEvent> { e ->
    val packet = e.packet()
    if (packet !is S2CEntityEquipmentUpdatePacket) return@on

    for (slot in packet.equipment()) {
        log.info("${slot.slot()} = ${slot.item()} x${slot.count()}")
    }
}
```

`slot()` uses the vanilla names — `mainhand`, `offhand`, `head`, `chest`, `legs`, `feet`, `body`, `saddle`. This is how you see someone switch weapons the moment they do it, without waiting for a swing.

The same shape elsewhere:

| Packet | The list | One entry |
|---|---|---|
| `S2CEntityEquipmentUpdatePacket` | `equipment()` | `slot()`, `item()`, `count()` |
| `S2CInventoryPacket` | `contents()` | `item()`, `count()` |
| `S2CEntityAttributesPacket` | `attributes()` | `attribute()`, `base()`, `modifiers()` |
| `S2CSetTradeOffersPacket` | `offers()` | what the villager takes and gives, plus `uses()`, `maxUses()`, `disabled()` |
| `S2CStatisticsPacket` | `stats()` | `stat()`, `value()` |
| `S2CEntityTrackerUpdatePacket` | `values()` | `id()`, `value()` |

Two caveats. Items here are an id and a count, not full [items](../game/inventory.md) — a packet is what came over the wire, so there are no enchantments or lore on it; look the slot or the entity up in the world when you need those. And `S2CEntityTrackerUpdatePacket` gives you whatever the game tracked — pose, health, flags — rendered as text, with a numeric `id()` that means nothing on its own. It is good for noticing that something changed, not for parsing.

## Tied to the Minecraft version

Packets are the only part of the API tied to the game version. When Minecraft updates, packet names and fields can change. If your script uses them, ship it alongside the matching client version and replace the packets jar in `.sdk/` when you update.

Scripts that never touch packets survive a Minecraft update untouched.
