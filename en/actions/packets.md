# Packets

A packet event hands you one Minecraft protocol packet decoded into a Java record. Packet events fire on the network thread, not the client thread, and the record is a flat snapshot: only what its accessors expose survived the decode.

```kotlin
on<PacketReceiveEvent> { e ->
    val packet = e.packet()
    if (packet !is S2CGameMessagePacket || packet.overlay()) return@on
    if (!packet.content().contains("duel")) return@on

    onClientThread { chat.sendToServer("/next") }
}
```

## Two threads

| Method | Type | Description |
|---|---|---|
| `onClientThread(action)` | `Unit` | queues the action on the client thread, skipped when the script is off by then — [Timers and tasks](../extras/tasks.md#getting-onto-the-client-thread) |
| `client.onClientThread()` | `boolean` | true when the caller already runs on the client thread — [Timers and tasks](../extras/tasks.md#getting-onto-the-client-thread) |

Reading the record's accessors off the client thread is safe.
`interaction`, `raycast`, `rotations.apply`, inventory and container writes, `world.blockEntitiesIn` and `world.removeEntity` throw [`ScriptThreadException`](../extras/limits.md#exceptions) off it; `packets.send` returns `false`.

## Receiving

| Method | Type | Description |
|---|---|---|
| `PacketReceiveEvent.packet()` | `S2CPacket` | decoded clientbound packet, never null in a handler — [Event list](../events/reference.md) |
| `PacketSendEvent.packet()` | `C2SPacket` | decoded serverbound packet, never null in a handler — [Event list](../events/reference.md) |

Only packets the client has a decoder for reach a handler; anything else fires nothing, and a decode failure is logged once per packet class.
Inside a bundle each sub-packet fires separately.

### Packet, S2CPacket, C2SPacket

| Method | Type | Description |
|---|---|---|
| `Packet` | `interface` | root marker of every packet record, declares no members |
| `S2CPacket` | `interface` | server-to-client marker, extends `Packet` |
| `C2SPacket` | `interface` | client-to-server marker, extends `Packet`; the type `send` takes |

`nursultan.packet.*`, `nursultan.packet.c2s.*` and `nursultan.packet.s2c.*` are default imports, so record and enum names need no `import` line.

## Cancelling

| Method | Type | Description |
|---|---|---|
| `PacketReceiveEvent.cancel()` | `void` | Minecraft never handles that packet |
| `PacketSendEvent.cancel()` | `void` | the packet never leaves the client |

Cancelling a received sub-packet inside a bundle removes it from the bundle.
Both events ignore `EventOptions` — `priority` and `ignoreCancelled` alike.

## Sending

| Method | Type | Description |
|---|---|---|
| `packets.send(packet)` | `boolean` | sends the record as a vanilla packet; false off the client thread, without a player or network handler, on a null or observe-only record, or when a field could not be converted (main thread only) |
| `packets.sendSequenced(packet)` | `boolean` | sends through the interaction manager, which fills in the server sequence number; same false conditions plus a missing world or interaction manager (main thread only) |

`packets` is `game.packets()`; a refused send warns once per packet class in the script console. `send` reuses the record's own `sequence` field, `sendSequenced` replaces it.
A packet sent from a script goes out through the same connection call, so it fires `PacketSendEvent` again.

## Fidelity

| Minecraft type | In the record | Note |
|---|---|---|
| `ItemStack` | `String` + `int` | registry id and count; enchantments, components and NBT are gone |
| `Text` | `String` | plain text from `getString()`, no colours, styles or click events |
| `Identifier` | `String` | `namespace:path` |
| `UUID` | `String` | `toString()` |
| `NbtCompound` | `String` | SNBT text, `""` when the packet carried none |
| enum constant | script enum | matched by constant name — [Packet enums](packets/enums.md) |

A vanilla constant with no script twin decodes to `null` and logs one warning; a script constant with no vanilla twin makes the whole send fail.
Full items, components and signatures are reachable only through the world and [Inventory and items](../game/inventory.md), never off a packet.

## The reference

| Page | Contents |
|---|---|
| [Packets you can send](packets/c2s.md) | 65 client-to-server records; 51 sendable, 14 observe-only |
| [Entity packets](packets/s2c-entities.md) | spawning, movement, rotation, velocity, damage, effects, equipment, attributes, tracked data |
| [World packets](packets/s2c-world.md) | blocks, chunks, light, world border, time, weather, sounds, particles, explosions |
| [Screen and chat packets](packets/s2c-screens.md) | screen handlers, inventories, trades, recipe book, chat, titles, tab list, scoreboard |
| [Packet enums](packets/enums.md) | 34 enums, 180 constants used as record field types |

Packet records mirror the Minecraft protocol of the client they ship with (1.21.11) — the only part of the API tied to the game version; a Minecraft update can rename or reshape a record.
The client rewrites `.sdk/` on every launch, so the packets jar follows the client you develop against.
