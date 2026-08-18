# World packets

Server-to-client records about blocks, chunks, light, the world border, time, weather, sounds, particles and explosions. Every one arrives through `PacketReceiveEvent.packet()`; positions are in blocks, chunk coordinates are blocks/16, rotations are degrees and ticks run 20 per second — threads, cancelling and what a decode drops are on [Packets](../packets.md).

```kotlin
on<PacketReceiveEvent> { e ->
    val packet = e.packet()
    if (packet !is S2CBlockUpdatePacket) return@on

    // a block somewhere turned into air
    if (packet.block() == "minecraft:air") log("${packet.x()} ${packet.y()} ${packet.z()}")
}
```

## S2CBlockBreakingProgressPacket

Break-progress overlay for a block another entity is mining.

| Component | Type | Description |
|---|---|---|
| `entityId()` | `int` | network id of the mining entity |
| `x()` | `int` | block x |
| `y()` | `int` | block y |
| `z()` | `int` | block z |
| `progress()` | `int` | break stage 0..9, anything outside clears the overlay |

## S2CBlockEntityUpdatePacket

Block entity data for one position.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | block x |
| `y()` | `int` | block y |
| `z()` | `int` | block z |
| `type()` | `String` | namespaced block entity type id |
| `nbt()` | `String` | SNBT payload, `""` when the packet carried none |

## S2CBlockEventPacket

Block action — note block ping, chest lid, piston push, bell ring.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | block x |
| `y()` | `int` | block y |
| `z()` | `int` | block z |
| `type()` | `int` | block-specific action id |
| `data()` | `int` | block-specific action parameter |
| `block()` | `String` | namespaced id of the block acted on |

## S2CBlockUpdatePacket

Single block change; only the block id survives, not the state properties.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | block x |
| `y()` | `int` | block y |
| `z()` | `int` | block z |
| `block()` | `String` | namespaced id of the new block |

## S2CBlockValueDebugPacket

Debug block-value push; the value payload is not exposed.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | block x |
| `y()` | `int` | block y |
| `z()` | `int` | block z |

## S2CChunkBiomeDataPacket

Biome-only chunk resend; no components.

## S2CChunkDataPacket

Full chunk send; sections, block entities, heightmaps and light are not exposed.

| Component | Type | Description |
|---|---|---|
| `chunkX()` | `int` | chunk x, blocks/16 |
| `chunkZ()` | `int` | chunk z, blocks/16 |

## S2CChunkDeltaUpdatePacket

Several block changes inside one 16x16x16 chunk section.

| Component | Type | Description |
|---|---|---|
| `blocks()` | `List<BlockChange>` | changed blocks, positions already absolute |

### S2CChunkDeltaUpdatePacket.BlockChange

One block change inside a chunk-delta update.

| Component | Type | Description |
|---|---|---|
| `x()` | `int` | absolute block x |
| `y()` | `int` | absolute block y |
| `z()` | `int` | absolute block z |
| `block()` | `String` | namespaced id of the new block |
| `air()` | `boolean` | the new state is air |

## S2CChunkLoadDistancePacket

Server-side view distance change.

| Component | Type | Description |
|---|---|---|
| `distance()` | `int` | view distance in chunks |

## S2CChunkRenderDistanceCenterPacket

New center chunk for chunk loading.

| Component | Type | Description |
|---|---|---|
| `chunkX()` | `int` | center chunk x |
| `chunkZ()` | `int` | center chunk z |

## S2CChunkSentPacket

End of a chunk batch; answered with `C2SAcknowledgeChunksPacket`.

| Component | Type | Description |
|---|---|---|
| `batchSize()` | `int` | number of chunks in the finished batch |

## S2CChunkValueDebugPacket

Debug chunk-value push; the value payload is not exposed.

| Component | Type | Description |
|---|---|---|
| `chunkX()` | `int` | chunk x |
| `chunkZ()` | `int` | chunk z |

## S2CDifficultyPacket

World difficulty sync.

| Component | Type | Description |
|---|---|---|
| `difficulty()` | `Difficulty?` | current difficulty, null when the constant is unmapped |
| `difficultyLocked()` | `boolean` | difficulty is locked |

## S2CExplosionPacket

Explosion effect; particle type, sound and the destroyed-block list are not exposed.

| Component | Type | Description |
|---|---|---|
| `x()` | `double` | explosion center x in blocks |
| `y()` | `double` | explosion center y in blocks |
| `z()` | `double` | explosion center z in blocks |
| `radius()` | `float` | explosion radius in blocks |
| `blockCount()` | `int` | number of blocks destroyed |
| `knockbackX()` | `double` | player knockback x in blocks/tick, 0 when absent |
| `knockbackY()` | `double` | player knockback y in blocks/tick, 0 when absent |
| `knockbackZ()` | `double` | player knockback z in blocks/tick, 0 when absent |

## S2CGameJoinPacket

Initial join info for the play phase.

| Component | Type | Description |
|---|---|---|
| `playerEntityId()` | `int` | the local player's entity network id |
| `hardcore()` | `boolean` | hardcore world |
| `dimensionIds()` | `List<String>` | namespaced ids of every dimension on the server |
| `maxPlayers()` | `int` | advertised player slot count |
| `viewDistance()` | `int` | server view distance in chunks |
| `simulationDistance()` | `int` | server simulation distance in chunks |
| `reducedDebugInfo()` | `boolean` | the F3 screen is reduced |
| `showDeathScreen()` | `boolean` | show the death screen instead of respawning at once |
| `doLimitedCrafting()` | `boolean` | crafting needs a recipe-book unlock |
| `enforcesSecureChat()` | `boolean` | server requires signed chat |

## S2CGameStateChangePacket

Game-state change value; the reason is not exposed, so the value alone is ambiguous.

| Component | Type | Description |
|---|---|---|
| `value()` | `float` | reason-specific: gamemode id, rain gradient 0..1, demo screen id |

## S2CGameTestHighlightPosPacket

Game-test debug position highlight.

| Component | Type | Description |
|---|---|---|
| `absoluteX()` | `int` | world-space x |
| `absoluteY()` | `int` | world-space y |
| `absoluteZ()` | `int` | world-space z |
| `relativeX()` | `int` | x relative to the test structure origin |
| `relativeY()` | `int` | y relative to the test structure origin |
| `relativeZ()` | `int` | z relative to the test structure origin |

## S2CLightUpdatePacket

Chunk light data; the light arrays are not exposed.

| Component | Type | Description |
|---|---|---|
| `chunkX()` | `int` | chunk x |
| `chunkZ()` | `int` | chunk z |

## S2CMapUpdatePacket

Filled-map update; pixel colors, icons and the updated rectangle are not exposed.

| Component | Type | Description |
|---|---|---|
| `mapId()` | `int` | numeric map id |
| `scale()` | `byte` | zoom level 0..4, 0 is one block per pixel |
| `locked()` | `boolean` | locked in a cartography table |

## S2CParticlePacket

Particle spawn request; the particle type and its parameters are not exposed.

| Component | Type | Description |
|---|---|---|
| `x()` | `double` | spawn center x in blocks |
| `y()` | `double` | spawn center y in blocks |
| `z()` | `double` | spawn center z in blocks |
| `offsetX()` | `float` | gaussian spread on x in blocks |
| `offsetY()` | `float` | gaussian spread on y in blocks |
| `offsetZ()` | `float` | gaussian spread on z in blocks |
| `speed()` | `float` | particle speed multiplier |
| `count()` | `int` | particle count; 0 turns the offsets into a velocity |
| `forceSpawn()` | `boolean` | ignores the client's particle distance limit |
| `important()` | `boolean` | spawns even on the lowest particle setting |

## S2CPlaySoundFromEntityPacket

Sound anchored to an entity.

| Component | Type | Description |
|---|---|---|
| `sound()` | `String` | namespaced sound event id |
| `category()` | `SoundCategory?` | mixer channel, null when the constant is unmapped |
| `entityId()` | `int` | source entity network id |
| `volume()` | `float` | volume multiplier, 1.0 is nominal |
| `pitch()` | `float` | pitch multiplier, 1.0 is nominal |
| `seed()` | `long` | random seed picking the sound variant |

## S2CPlaySoundPacket

Positional sound.

| Component | Type | Description |
|---|---|---|
| `sound()` | `String` | namespaced sound event id |
| `category()` | `SoundCategory?` | mixer channel, null when the constant is unmapped |
| `x()` | `double` | sound x in blocks, wire value is 1/8-block fixed point |
| `y()` | `double` | sound y in blocks |
| `z()` | `double` | sound z in blocks |
| `volume()` | `float` | volume multiplier, 1.0 is nominal |
| `pitch()` | `float` | pitch multiplier, 1.0 is nominal |
| `seed()` | `long` | random seed picking the sound variant |

## S2CSimulationDistancePacket

Server simulation distance change.

| Component | Type | Description |
|---|---|---|
| `simulationDistance()` | `int` | simulation distance in chunks |

## S2CStartChunkSendPacket

Start of a chunk batch; no components.

## S2CStopSoundPacket

Stops playing sounds.

| Component | Type | Description |
|---|---|---|
| `soundId()` | `String?` | namespaced sound event id, null stops the whole category |
| `category()` | `SoundCategory?` | mixer channel, null means every category |

## S2CTestInstanceBlockStatusPacket

Test instance block status report.

| Component | Type | Description |
|---|---|---|
| `status()` | `String` | plain-text status message |
| `sizeX()` | `Integer?` | test area size x in blocks, null when the packet carries no size |
| `sizeY()` | `Integer?` | test area size y in blocks, null when the packet carries no size |
| `sizeZ()` | `Integer?` | test area size z in blocks, null when the packet carries no size |

## S2CTickStepPacket

How many ticks to step while the world is frozen.

| Component | Type | Description |
|---|---|---|
| `tickSteps()` | `int` | remaining ticks to step |

## S2CUnloadChunkPacket

Chunk removed from the client world.

| Component | Type | Description |
|---|---|---|
| `chunkX()` | `int` | chunk x |
| `chunkZ()` | `int` | chunk z |

## S2CUpdateTickRatePacket

Server tick rate and freeze state.

| Component | Type | Description |
|---|---|---|
| `tickRate()` | `float` | ticks per second, vanilla default 20 |
| `isFrozen()` | `boolean` | world ticking is frozen |

## S2CWaypointPacket

Locator-bar waypoint change; the waypoint id and its data are not exposed.

| Component | Type | Description |
|---|---|---|
| `operation()` | `WaypointOperation?` | track, update or untrack, null when unmapped |

## S2CWorldBorderCenterChangedPacket

World border center moved.

| Component | Type | Description |
|---|---|---|
| `centerX()` | `double` | border center x in blocks |
| `centerZ()` | `double` | border center z in blocks |

## S2CWorldBorderInitializePacket

Full world border state, sent on join.

| Component | Type | Description |
|---|---|---|
| `centerX()` | `double` | border center x in blocks |
| `centerZ()` | `double` | border center z in blocks |
| `size()` | `double` | current border diameter in blocks |
| `sizeLerpTarget()` | `double` | target border diameter in blocks |
| `sizeLerpTime()` | `long` | interpolation duration in milliseconds |
| `maxRadius()` | `int` | absolute world border limit in blocks |
| `warningBlocks()` | `int` | warning overlay distance in blocks |
| `warningTime()` | `int` | warning overlay lead time in seconds |

## S2CWorldBorderInterpolateSizePacket

World border starts resizing over time.

| Component | Type | Description |
|---|---|---|
| `size()` | `double` | current diameter in blocks |
| `sizeLerpTarget()` | `double` | target diameter in blocks |
| `sizeLerpTime()` | `long` | interpolation duration in milliseconds |

## S2CWorldBorderSizeChangedPacket

World border resized instantly.

| Component | Type | Description |
|---|---|---|
| `sizeLerpTarget()` | `double` | new diameter in blocks |

## S2CWorldBorderWarningBlocksChangedPacket

World border warning distance changed.

| Component | Type | Description |
|---|---|---|
| `warningBlocks()` | `int` | warning overlay distance in blocks |

## S2CWorldBorderWarningTimeChangedPacket

World border warning lead time changed.

| Component | Type | Description |
|---|---|---|
| `warningTime()` | `int` | warning overlay lead time in seconds |

## S2CWorldEventPacket

World event — block break sound and particles, door, portal, dispenser.

| Component | Type | Description |
|---|---|---|
| `eventId()` | `int` | vanilla WorldEvents id |
| `x()` | `int` | event block x |
| `y()` | `int` | event block y |
| `z()` | `int` | event block z |
| `data()` | `int` | event-specific parameter, often a packed block state id |
| `global()` | `boolean` | audible across the dimension regardless of distance |

## S2CWorldTimeUpdatePacket

World time sync.

| Component | Type | Description |
|---|---|---|
| `time()` | `long` | total world age in ticks |
| `timeOfDay()` | `long` | time of day in ticks, 0..23999 per day |
| `tickDayTime()` | `boolean` | the daylight cycle is advancing (`doDaylightCycle`) |
