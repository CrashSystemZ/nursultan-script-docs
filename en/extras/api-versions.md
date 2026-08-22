# API versions

`ApiVersion.CURRENT` is 4. `requireApi(n)` fails the script at load when the running client is older; it cannot rescue a compile error, because a name that does not exist on the older SDK never compiles in the first place.

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
| `ApiVersion.CURRENT` | `int` | script API version of this client, currently 4 |
| `ApiVersion.require(minimum)` | `void` | static (throws `ScriptApiException` when `CURRENT` < `minimum`) |
| `requireApi(minimum)` | `Unit` | the DSL form of `ApiVersion.require` (throws `ScriptApiException` when `CURRENT` < `minimum`) |

`ApiVersion` has a private constructor: there is no instance, only the two static members.
The client appends `this client provides v4` to every `Unresolved reference` compile error, and compilation happens before the first line runs.
Nothing is gated per member: every addition above is present unconditionally in a client of that version, and `requireApi(n)` is the only check that exists.
Packet records follow the Minecraft version, not this number — see [Packets](../actions/packets.md).

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
