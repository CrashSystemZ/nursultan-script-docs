# API versions

`ApiVersion.CURRENT` is 2. `requireApi(n)` fails the script at load when the running client is older; it cannot rescue a compile error, because a name that does not exist on the older SDK never compiles in the first place.

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
| `ApiVersion.CURRENT` | `int` | script API version of this client, currently 2 |
| `ApiVersion.require(minimum)` | `void` | static (throws `ScriptApiException` when `CURRENT` < `minimum`) |
| `requireApi(minimum)` | `Unit` | the DSL form of `ApiVersion.require` (throws `ScriptApiException` when `CURRENT` < `minimum`) |

`ApiVersion` has a private constructor: there is no instance, only the two static members.
The client appends `this client provides v2` to every `Unresolved reference` compile error, and compilation happens before the first line runs.

## What is new in 2

| Added in 2 | Documented on |
|---|---|
| `Client.user()`, `assets()`, `clipboard()`, `gpu()`; `Game.recipes()`, `settings()` | [How a script works](../start/lifecycle.md) |
| `Assets` — the `assets` root | [The assets folder](assets.md) |
| `base64(...)`, `base64Encode(...)`, `font(name, bytes)`, `image(name, bytes)` | [The assets folder](assets.md) |
| `Clipboard` — the `clipboard` root | [Messages](../ui/messages.md) |
| `User`, `DiscordUser` — the `user` root | [Your account](user.md) |
| `Config.getList`, `getIntList`, `getDoubleList`, `putList`, `putIntList`, `putDoubleList` | [Saving data](../settings/storage.md) |
| `Hunger` — `player.hunger()` | [Your player](../game/player.md) |
| `SelfPlayer.attackCooldown(tickDelta)`, `cooldownPeriod()`, `ticksSinceLastAttack()` | [Your player](../game/player.md) |
| `LivingEntity.swinging()`, `swingTicks()` | [Entities and filters](../game/entities.md) |
| `GameSettings`, `Perspective` — the `gameSettings` root | [Movement](../actions/control.md) |
| `Recipes`, `RecipeEntry`, `RecipeKind` — the `recipes` root | [Containers](../game/containers.md) |
| `Container.Batch.onFinish`, `Inventory.Batch.onFinish` | [Containers](../game/containers.md), [Inventory and items](../game/inventory.md) |
| `Item.buildable()` | [Inventory and items](../game/inventory.md) |
| `Item.cooldownProgress()`, `cooldownProgress(tickDelta)`, `setCooldown(ticks)`, `removeCooldown()` | [Inventory and items](../game/inventory.md) |
| `TabEntry.pingMs(value)` | [Server, scoreboard, tab list](../game/server.md) |
| `input(name, value, placeholder)` | [Kinds of settings](../settings/types.md) |
| `Gpu` and the 11 GPU types | [Your own geometry](../ui/gpu.md) |
| `Weight` and the weight overloads on `Render` | [2D render](../ui/render-2d.md) |
| `blur(x, y, width, height, radius, argb, tl, tr, bl, br)` | [2D render](../ui/render-2d.md) |
| `Shader.setMat4`, `Shader.set(uniform, texture, filter, wrap)` | [Shaders](../ui/shaders.md) |
| `EventOptions(ignoreCancelled)` | [Subscribing](../events/basics.md) |
| `FireworkEntitySpeedEvent` | [Event list](../events/reference.md) |
| `PerspectiveEvent` | [Event list](../events/reference.md) |
| `RenderCrosshairEvent` | [Event list](../events/reference.md) |
| `Render3DEvent.viewMatrix()`, `projectionMatrix()` | [Event list](../events/reference.md) |

Nothing is gated per member: every addition above is present unconditionally in a v2 client, and `requireApi(2)` is the only check that exists.
Packet records follow the Minecraft version, not this number — see [Packets](../actions/packets.md).

## Things that no longer do anything

`@NoEffect` marks an element that still exists and still compiles but whose value the client never reads.

| Method | Type | Description |
|---|---|---|
| `NoEffect.value()` | `String` | sentence saying why the element is never read |

`@Retention(CLASS)` — visible in the IDE and in the SDK jar, never through reflection.
Targets: type, method, field, parameter, record component.

| Member | Type | Description |
|---|---|---|
| `Priority` | `enum` | whole type marked, five constants kept so old scripts compile (no effect: every script handler runs at the same moment) |
| `EventOptions.priority` | `Priority` | record component and its accessor (deprecated) (no effect: every script handler runs at the same moment) |
| `EventOptions.priority(priority)` | `EventOptions` | copy of `DEFAULT` carrying the value (deprecated) (no effect: every script handler runs at the same moment) |
| `ScriptScope.on<E>(priority, ignoreCancelled) { }` | `Subscription` | discards the argument (deprecated, drop the argument) |
| `ScriptScope.on(type, priority, ignoreCancelled) { }` | `Subscription` | discards the argument (deprecated, drop the argument) |
| `EntryScope.on<E>(priority, ignoreCancelled) { }` | `Subscription` | discards the argument (deprecated, drop the argument) |
| `EntryScope.on(type, priority, ignoreCancelled) { }` | `Subscription` | discards the argument (deprecated, drop the argument) |
| `RotationOptions.clientSide` | `boolean` | record component and its accessor (deprecated) (no effect: the rotation always reaches the server) |
| `RotationOptions.clientSide(value)` | `RotationOptions` | copy carrying the value (deprecated) (no effect: the rotation always reaches the server) |
| `RotationOptions.normalizeMouseMovement` | `boolean` | record component and its accessor (deprecated) (no effect: the angle is always snapped to a mouse step) |
| `RotationOptions.normalizeMouseMovement(value)` | `RotationOptions` | copy carrying the value (deprecated) (no effect: the angle is always snapped to a mouse step) |

All eleven still compile and still store what you give them; nothing reads the stored value.
The order that replaced `Priority` is on [Subscribing](../events/basics.md#priority-does-nothing), the two rotation flags on [Rotations](../actions/rotations.md#options).

## What a bump costs

The SDK api jar is named `nursultan-script-api-v<N>.jar`, so an update drops in a new file and rewrites `build.gradle.kts` to point at it; IDEA re-syncs the project once.
`ApiVersion.CURRENT` is part of the compile-cache key, so a bump invalidates every cached compiled script and forces a full recompile on the first launch after the update.
