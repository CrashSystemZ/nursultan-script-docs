# API versions

`ApiVersion.CURRENT` is 5. `requireApi(n)` fails the script at load when the running client is older; it cannot rescue a compile error, because a name that does not exist on the older SDK never compiles in the first place.

```kotlin
requireApi(2)

name("Mesh demo")

// gpu arrived in API 2; on a v1 client this file refuses to load
val format = gpu.format(VertexAttribute.floats(3), VertexAttribute.color())
val mesh = gpu.indexedMesh(format)
```

## Requiring a version

| Method | Type | Description |
|---|---|---|
| `ApiVersion.CURRENT` | `int` | script API version of this client, currently 5 |
| `ApiVersion.require(minimum)` | `void` | static (throws `ScriptApiException` when `CURRENT` < `minimum`) |
| `requireApi(minimum)` | `Unit` | the DSL form of `ApiVersion.require` (throws `ScriptApiException` when `CURRENT` < `minimum`) |

`ApiVersion` has a private constructor: no instance, only the two static members, and the client appends `this client provides v5` to every `Unresolved reference` compile error.
Packet records follow the Minecraft version, not this number — see [Packets](../actions/packets.md).

## What each version added

### API 3

| Added in 3 | Documented on |
|---|---|
| `combat.explosionExposure(target, source)` | [Interaction](../actions/interaction.md) |
| `combat.explosionDamage(target, source, power)` | [Interaction](../actions/interaction.md) |
| `combat.explosionDamageTaken(target, source, power)` | [Interaction](../actions/interaction.md) |
| `combat.damageAfterArmor(target, damage)` | [Interaction](../actions/interaction.md) |
| `living.visibleEffects()` | [Entities and filters](../game/entities.md) |
| `block.collisionBoxes()` | [World and blocks](../game/world.md) |
| `block.outlineBoxes()` | [World and blocks](../game/world.md) |
| `world.blockCollisionsIn(box)` | [World and blocks](../game/world.md) |
| `world.isBlockSpaceFree(box)` | [World and blocks](../game/world.md) |
| `RenderItemEvent.translate(x, y, z)` | [Event list](../events/reference.md) |
| `RenderItemEvent.rotate(degrees, axisX, axisY, axisZ)` | [Event list](../events/reference.md) |
| `RenderItemEvent.rotateX(degrees)` | [Event list](../events/reference.md) |
| `RenderItemEvent.rotateY(degrees)` | [Event list](../events/reference.md) |
| `RenderItemEvent.rotateZ(degrees)` | [Event list](../events/reference.md) |
| `RenderItemEvent.scale(x, y, z)` | [Event list](../events/reference.md) |
| `RenderItemEvent.Matrix` | [Event list](../events/reference.md) |

### API 4

| Added in 4 | Documented on |
|---|---|
| `client.party()` | [How a script works](../start/lifecycle.md) |
| `party` | [Party messages](party.md) |
| `Party` | [Party messages](party.md) |
| `PartyChannel` | [Party messages](party.md) |
| `PartyMember` | [Party messages](party.md) |
| `PartyMessage` | [Party messages](party.md) |
| `PartyShapedMessage` | [Party messages](party.md) |
| `PartyMessageKind` | [Party messages](party.md) |
| `PartyFields` | [Party messages](party.md) |
| `PartyShape` | [Party messages](party.md) |
| `PartyShapeBuilder` | [Party messages](party.md) |
| `PartyField` | [Party messages](party.md) |
| `PartyFieldType` | [Party messages](party.md) |
| `PartyWire` | [Party messages](party.md) |
| `PartyStruct` | [Party messages](party.md) |
| `PartyShapedWriter` | [Party messages](party.md) |
| `PartyPayloadWriter` | [Party messages](party.md) |
| `PartyPayloadReader` | [Party messages](party.md) |
| `PartyTarget` | [Party messages](party.md) |
| `PartyTargetKind` | [Party messages](party.md) |
| `SenderRule` | [Party messages](party.md) |
| `PartySendResult` | [Party messages](party.md) |
| 43 `nursultan.dsl` party helpers — `shape`, the field builders, `send`, `publish`, the typed field readers | [Party messages](party.md) |
| `nursultan.party.*` as a default import | [Party messages](party.md) |
| `entity.isItem()` | [Entities and filters](../game/entities.md) |
| `entity.asItemEntity()` | [Entities and filters](../game/entities.md) |
| `ItemEntity` | [Entities and filters](../game/entities.md) |

### API 5

| Added in 5 | Documented on |
|---|---|
| `party.code()` | [Party messages](party.md) |
| `PartyMember.color()` | [Party messages](party.md) |
| `PartyMember.position()` | [Party messages](party.md) |
| `PartyMember.positionAge()` | [Party messages](party.md) |
| `player.serverSprinting()` | [Your player](../game/player.md) |
| `player.velocity(value)` | [Your player](../game/player.md) |
| `render.pushScissor(x, y, width, height)` | [2D render](../ui/render-2d.md) |
| `render.popScissor()` | [2D render](../ui/render-2d.md) |

Nothing is gated per member: every addition above is present unconditionally in a client of that version, and `requireApi(n)` is the only check that exists.
API 1 is the surface that carries no marker at all; API 2 members are marked `(API 2)` in the tables of the page that documents them.

## Things that no longer do anything

`@NoEffect` marks an element that still exists and still compiles but whose value the client never reads.

| Method | Type | Description |
|---|---|---|
| `NoEffect.value()` | `String` | sentence saying why the element is never read |

`@Retention(CLASS)` — visible in the IDE and in the SDK jar, never through reflection.
Targets: type, method, field, parameter, record component.

### What is marked

| Member | Type | Description |
|---|---|---|
| `Priority` | `enum` | whole type marked, five constants kept so old scripts compile (no effect on dispatch order) |
| `EventOptions.priority()` | `Priority` | record component and its accessor (deprecated) (no effect: the value is never read) |
| `EventOptions.priority(priority)` | `EventOptions` | copy of `DEFAULT` carrying the value (deprecated) (no effect: the value is never read) |
| `ScriptScope.on<E>(priority, ignoreCancelled) { }` | `Subscription` | discards the argument (deprecated, drop the argument) |
| `ScriptScope.on(type, priority, ignoreCancelled) { }` | `Subscription` | discards the argument (deprecated, drop the argument) |
| `EntryScope.on<E>(priority, ignoreCancelled) { }` | `Subscription` | discards the argument (deprecated, drop the argument) |
| `EntryScope.on(type, priority, ignoreCancelled) { }` | `Subscription` | discards the argument (deprecated, drop the argument) |
| `RotationOptions.clientSide()` | `boolean` | record component and its accessor (deprecated) (no effect: the rotation always reaches the server) |
| `RotationOptions.clientSide(value)` | `RotationOptions` | copy carrying the value (deprecated) (no effect: the rotation always reaches the server) |
| `RotationOptions.normalizeMouseMovement()` | `boolean` | record component and its accessor (deprecated) (no effect: the angle is always snapped to a mouse step) |
| `RotationOptions.normalizeMouseMovement(value)` | `RotationOptions` | copy carrying the value (deprecated) (no effect: the angle is always snapped to a mouse step) |

All eleven still compile and still store what you give them; nothing reads the stored value.
The order that replaced `Priority` is on [Subscribing](../events/basics.md#priority-does-nothing), the two rotation flags on [Rotations](../actions/rotations.md#options).

## What a bump costs

The SDK api jar is named `nursultan-script-api-v<N>.jar`, so an update drops in a new file and rewrites `build.gradle.kts` to point at it; IDEA re-syncs the project once.
`ApiVersion.CURRENT` is part of the compile-cache key, so a bump invalidates every cached compiled script and forces a full recompile on the first launch after the update.
