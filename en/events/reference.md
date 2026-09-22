# Event list

51 events, grouped by area. Every one fires on the Minecraft client thread except `PacketReceiveEvent` and `PacketSendEvent`. An event with no member table carries no payload. Subscribing, cancelling and options are on [Subscribing](basics.md).

```kotlin
on<BlockHitEvent> {
    // the top face of a block is never hit
    if (it.side() == Side.UP) it.cancel()
}
```

## Ticks

| Event | Cancellable | Fires when |
|---|---|---|
| `ClientTickEvent` | no | once per client tick, also with no world loaded |
| `PrePlayerTickEvent` | no | once per client tick, before the local player ticks |
| `PlayerTickEvent` | no | mid local-player tick, at the cooldown update |
| `PostPlayerTickEvent` | no | after the local-player tick finished |
| `TimerEvent` | no | once per render tick, before the tick duration is used |

### TimerEvent

| Method | Type | Description |
|---|---|---|
| `timer()` | `float` | client tick-rate multiplier, 1.0 = normal speed |
| `timer(value)` | `void` | overrides it; tick duration is divided by it, not clamped |

Reset to 1.0 before every dispatch, so it does not persist between frames; `0` divides by zero.

## Movement

| Event | Cancellable | Fires when |
|---|---|---|
| `MoveInputEvent` | yes (never read) | each tick, after the movement keys were sampled |
| `MoveEvent` | yes | before a client entity applies its movement vector |
| `MovePacketEvent` | yes | once per tick, before the position and rotation packets go out |
| `MoveRelativeEvent` | no | after movement input was turned into velocity |
| `TravelEvent` | yes | before the local player's vanilla physics step |
| `JumpEvent` | no | the local player jumps |
| `SlowdownEvent` | no | the game reads the item-use movement multiplier |
| `FireworkEntitySpeedEvent` | no | a rocket boosts a gliding entity, once per tick per rocket (API 2) |
| `BlockPushEvent` | yes | the local player is about to be pushed out of a block |
| `EntityPushEvent` | yes | the local player pushes or is pushed by an entity |
| `WebCollisionEvent` | yes | the local player collides with a cobweb |

### MoveInputEvent

| Method | Type | Description |
|---|---|---|
| `forward()` | `boolean` | forward key held |
| `forward(value)` | `void` | overrides the forward flag; the game re-reads it |
| `backward()` | `boolean` | backward key held |
| `backward(value)` | `void` | overrides the backward flag; the game re-reads it |
| `left()` | `boolean` | strafe-left key held |
| `left(value)` | `void` | overrides the left flag; the game re-reads it |
| `right()` | `boolean` | strafe-right key held |
| `right(value)` | `void` | overrides the right flag; the game re-reads it |
| `jump()` | `boolean` | jump key held |
| `jump(value)` | `void` | overrides the jump flag; the game re-reads it |
| `sneak()` | `boolean` | sneak key held |
| `sneak(value)` | `void` | overrides the sneak flag; the game re-reads it |
| `sprint()` | `boolean` | sprint key held |
| `sprint(value)` | `void` | overrides the sprint flag; also drives the sprint key state |
| `moving()` | `boolean` | true when forward, backward, left or right is set |
| `toInput()` | [`MoveInput`](../actions/prediction.md) | immutable snapshot of the seven flags |

The game rebuilds the tick's player input from all seven flags. Cancelling does not stop vanilla input — the mixin never reads the flag.

### MoveEvent

| Method | Type | Description |
|---|---|---|
| `movement()` | [`Vec`](../game/math.md) | movement delta in blocks for this move call |
| `movement(value)` | `void` | replaces the whole vector; the game re-reads it |
| `x()` | `double` | movement X in blocks |
| `y()` | `double` | movement Y in blocks |
| `z()` | `double` | movement Z in blocks |
| `set(x, y, z)` | `void` | replaces the vector component-wise; the game re-reads it |

Cancelling skips the whole move call: no displacement and no collision handling.

### MovePacketEvent

| Method | Type | Description |
|---|---|---|
| `x()` | `double` | outgoing packet X in world blocks |
| `x(value)` | `void` | overrides outgoing X; the sent packet re-reads it |
| `y()` | `double` | outgoing packet Y in world blocks, feet position |
| `y(value)` | `void` | overrides outgoing Y; the sent packet re-reads it |
| `z()` | `double` | outgoing packet Z in world blocks |
| `z(value)` | `void` | overrides outgoing Z; the sent packet re-reads it |
| `yaw()` | `float` | outgoing packet yaw in degrees, unclamped |
| `yaw(value)` | `void` | overrides outgoing yaw; the sent packet re-reads it |
| `pitch()` | `float` | outgoing packet pitch in degrees, unclamped |
| `pitch(value)` | `void` | overrides outgoing pitch; the sent packet re-reads it |
| `onGround()` | `boolean` | outgoing on-ground flag |
| `onGround(value)` | `void` | overrides the flag; the sent packet re-reads it |
| `sprinting()` | `boolean` | sprint state used by the sprint packet |
| `sprinting(value)` | `void` | overrides it; the sprint packet re-reads it |

Cancelling sends no position, rotation or sprint packet that tick.

### MoveRelativeEvent

| Method | Type | Description |
|---|---|---|
| `speed()` | `float` | movement speed factor applied to the input |
| `input()` | [`Vec`](../game/math.md) | raw input: x sideways, y upward, z forward, each about -1..1 |

Read-only: nothing written to this event is copied back.

### SlowdownEvent

| Method | Type | Description |
|---|---|---|
| `multiplier()` | `float` | item-use movement multiplier, 1.0 = no slowdown |
| `multiplier(value)` | `void` | overrides it; returned in place of the vanilla value, not clamped |

### FireworkEntitySpeedEvent

| Method | Type | Description |
|---|---|---|
| `speed()` | `double` | elytra boost factor, vanilla value 1.5 |
| `speed(value)` | `void` | overrides the boost for this tick; the game re-reads it |
| `firework()` | [`Entity`](../game/entities.md) | the firework rocket entity |
| `shooter()` | [`LivingEntity?`](../game/entities.md) | gliding entity being boosted, null when the rocket has none |

Fires for every rocket in the world, not only your own.

### WebCollisionEvent

| Method | Type | Description |
|---|---|---|
| `x()` | `int` | cobweb block X |
| `y()` | `int` | cobweb block Y |
| `z()` | `int` | cobweb block Z |
| `position()` | [`Vec`](../game/math.md) | block position as a vector, a copy |

Cancelling skips the cobweb slowdown.

## Combat

| Event | Cancellable | Fires when |
|---|---|---|
| `AttackEvent` | yes | before the client attacks an entity |
| `AttackedEvent` | no | after an attack went through |
| `TargetUpdateEvent` | no | the client's combat target changed or timed out |

### AttackEvent

| Method | Type | Description |
|---|---|---|
| `target()` | [`Entity`](../game/entities.md) | entity about to be attacked |

Cancelling aborts the attack entirely — no swing, no packet, and the attack cooldown is not reset.

### AttackedEvent

| Method | Type | Description |
|---|---|---|
| `target()` | [`Entity`](../game/entities.md) | entity that was just attacked |

### TargetUpdateEvent

| Method | Type | Description |
|---|---|---|
| `previous()` | [`Entity?`](../game/entities.md) | target before the change, null when there was none |
| `target()` | [`Entity?`](../game/entities.md) | new target, null when it was lost or timed out |
| `lost()` | `boolean` | true when `target()` is null |

Published by the client's target tracker: with no combat module running and no `combat.markTarget(...)` call it never fires.

## Blocks

| Event | Cancellable | Fires when |
|---|---|---|
| `BlockBreakingEvent` | yes | every frame the client processes the breaking input |
| `BlockHitEvent` | yes | the client starts or continues hitting a block |
| `BlockPlacedEvent` | no | a block is placed in the client world, by any placer |

### BlockBreakingEvent

| Method | Type | Description |
|---|---|---|
| `breaking()` | `boolean` | true while the attack key is held down |

Cancelling skips the whole breaking step for that frame, including the abort bookkeeping.

### BlockHitEvent

| Method | Type | Description |
|---|---|---|
| `x()` | `int` | block X in world block coordinates |
| `y()` | `int` | block Y in world block coordinates |
| `z()` | `int` | block Z in world block coordinates |
| `side()` | [`Side`](../game/raycast.md) | struck face |
| `position()` | [`Vec`](../game/math.md) | block position as a vector, a copy |

Fires twice per break flow — at the first hit and at each progress update; cancelling makes both return false, so no hit packet and no progress.

### BlockPlacedEvent

| Method | Type | Description |
|---|---|---|
| `block()` | [`Block`](../game/world.md) | placed block snapshot |
| `id()` | `String` | block registry id |
| `x()` | `int` | block X in world block coordinates |
| `y()` | `int` | block Y in world block coordinates |
| `z()` | `int` | block Z in world block coordinates |
| `position()` | [`Vec`](../game/math.md) | block position as a vector |

## Inventory

| Event | Cancellable | Fires when |
|---|---|---|
| `SlotClickEvent` | yes | a slot in a container screen was clicked |
| `RightClickEvent` | yes | the client runs its right-click step for a hand |
| `UseItemEvent` | yes | the client is about to send an item-use interaction |
| `StopUsingItemEvent` | yes | the client is about to stop using the held item |
| `FinishUsingItemEvent` | no | an item finished being used, food eaten, potion drunk |

### SlotClickEvent

| Method | Type | Description |
|---|---|---|
| `syncId()` | `int` | screen handler sync id of the open container |
| `slotId()` | `int` | slot index in the screen handler, -999 outside the window |
| `button()` | `int` | mouse button, 0 left and 1 right, or the target hotbar index for SWAP |
| `action()` | [`SlotAction`](../game/containers.md) | click action type, null maps to PICKUP |
| `item()` | [`Item`](../game/inventory.md) | stack in the clicked slot |

Cancelling drops the click: nothing is processed and no click packet is sent.

### RightClickEvent

| Method | Type | Description |
|---|---|---|
| `hand()` | [`Hand`](../game/inventory.md) | hand being used, null maps to MAIN_HAND |

Cancelling skips the whole use step — block and entity interaction included.

### UseItemEvent

| Method | Type | Description |
|---|---|---|
| `hand()` | [`Hand`](../game/inventory.md) | hand performing the interaction, null maps to MAIN_HAND |

Cancelling makes the interaction pass, so no item-use packet is sent.

### FinishUsingItemEvent

| Method | Type | Description |
|---|---|---|
| `item()` | [`Item`](../game/inventory.md) | stack whose use just completed |

## Input

| Event | Cancellable | Fires when |
|---|---|---|
| `KeyEvent` | yes | a key or mouse button goes down, up or repeats |
| `CharEvent` | yes | a character is typed into the window |
| `MouseScrollEvent` | yes | the mouse wheel scrolls |
| `LookInputEvent` | yes | an entity's look direction changes from cursor movement |

### KeyEvent

| Method | Type | Description |
|---|---|---|
| `key()` | [`Key`](../actions/keys.md) | pressed key or mouse button, `Key.UNKNOWN` when unmapped |
| `mods()` | `int` | GLFW modifier bitmask, 0 for synthetic events |
| `action()` | [`KeyAction`](../actions/keys.md) | PRESS, RELEASE or REPEAT |
| `mouse()` | `boolean` | true for mouse buttons |
| `pressed()` | `boolean` | true when the action is PRESS |
| `released()` | `boolean` | true when the action is RELEASE |
| `matches(other)` | `boolean` | true when the key constant is the same |

Cancelling keeps the key out of MC keybinds and screens.

### CharEvent

| Method | Type | Description |
|---|---|---|
| `codePoint()` | `int` | Unicode code point of the typed character |
| `character()` | `String` | the code point as a string, 1 or 2 chars |
| `mods()` | `int` | GLFW modifier bitmask active while typing |

### MouseScrollEvent

| Method | Type | Description |
|---|---|---|
| `vertical()` | `double` | vertical wheel delta, positive is up |
| `horizontal()` | `double` | horizontal wheel delta, 0 without a tilt wheel |

### LookInputEvent

| Method | Type | Description |
|---|---|---|
| `deltaX()` | `double` | horizontal cursor delta, multiplied by 0.15 to get yaw degrees |
| `deltaX(value)` | `void` | overrides the horizontal delta; the game re-reads it |
| `deltaY()` | `double` | vertical cursor delta, multiplied by 0.15 to get pitch degrees |
| `deltaY(value)` | `void` | overrides the vertical delta; the game re-reads it |

Fires for any entity, not only the local player; cancelling leaves yaw and pitch untouched.

## World and entities

| Event | Cancellable | Fires when |
|---|---|---|
| `EntitySpawnEvent` | no | an entity was added to the client world |
| `EntityRemoveEvent` | no | an entity is removed from the client world |
| `EffectEvent` | no | a status effect on the local player is added, refreshed or removed |
| `PotionImpactEvent` | no | a thrown potion collided, once per potion entity |
| `PlaySoundEvent` | yes | a positional world sound is about to be built |
| `WorldLoadEvent` | no | the client joins a world, respawns or changes dimension |

### EntitySpawnEvent

| Method | Type | Description |
|---|---|---|
| `entity()` | [`Entity`](../game/entities.md) | entity just added to the world |

### EntityRemoveEvent

| Method | Type | Description |
|---|---|---|
| `entity()` | [`Entity`](../game/entities.md) | entity being removed from the world |

Covers despawn, death removal and chunk unload alike; the reason is not exposed.

### EffectEvent

| Method | Type | Description |
|---|---|---|
| `effect()` | [`Effect?`](../game/entities.md) | effect snapshot, null only for `PLAYER_INIT` |
| `action()` | `EffectEvent.Action` | which map operation triggered the event |
| `added()` | `boolean` | true when the action is `ADD` |
| `removed()` | `boolean` | true when the action is `REMOVE` |

### EffectEvent.Action

| Constant | Description |
|---|---|
| `ADD` | effect newly inserted into the player's effect map |
| `UPDATE` | effect key already present, instance replaced |
| `REMOVE` | effect removed from the map |
| `PLAYER_INIT` | the local player's effect map was created, `effect()` is null |

### PotionImpactEvent

| Method | Type | Description |
|---|---|---|
| `potion()` | [`Entity`](../game/entities.md) | the potion projectile entity |
| `hitEntity()` | [`Entity?`](../game/entities.md) | entity that was hit, null for block and miss hits |
| `position()` | [`Vec?`](../game/math.md) | impact position in world blocks, null without a hit result |

### PlaySoundEvent

| Method | Type | Description |
|---|---|---|
| `id()` | `String` | sound registry id, empty when unresolvable |
| `category()` | `String` | lowercased sound category, empty when absent |
| `volume()` | `float` | sound volume, 1.0 = normal |
| `volume(value)` | `void` | overrides volume; the sound instance re-reads it |
| `pitch()` | `float` | pitch multiplier, vanilla range 0.5..2.0 |
| `pitch(value)` | `void` | overrides pitch; the sound instance re-reads it |
| `x()` | `double` | sound source X in world blocks |
| `x(value)` | `void` | overrides source X; the sound instance re-reads it |
| `y()` | `double` | sound source Y in world blocks |
| `y(value)` | `void` | overrides source Y; the sound instance re-reads it |
| `z()` | `double` | sound source Z in world blocks |
| `z(value)` | `void` | overrides source Z; the sound instance re-reads it |
| `position()` | [`Vec`](../game/math.md) | current x, y and z as a copy |
| `position(value)` | `void` | sets x, y and z from the vector |

`id()` and `category()` are read-only. Only sounds routed through the positional world-sound path fire this event; cancelling means the sound is not played.

## Packets

| Event | Cancellable | Fires when |
|---|---|---|
| `PacketReceiveEvent` | yes | a clientbound packet was decoded, before the game handles it |
| `PacketSendEvent` | yes | a serverbound packet is about to leave the client |

### PacketReceiveEvent

| Method | Type | Description |
|---|---|---|
| `packet()` | [`S2CPacket`](../actions/packets.md) | decoded clientbound packet |

Runs on the netty IO thread; inside a bundle each sub-packet fires separately and cancelling removes it from the bundle.

### PacketSendEvent

| Method | Type | Description |
|---|---|---|
| `packet()` | [`C2SPacket`](../actions/packets.md) | decoded serverbound packet |

Runs on the calling thread. For both events only packets the client can decode reach a handler, and `EventOptions` is ignored.

## Rendering

| Event | Cancellable | Fires when |
|---|---|---|
| `Render2DEvent` | no | every frame, in the script 2D pass after the HUD |
| `Render3DEvent` | no | every frame, during the world render |
| `CameraEvent` | yes | every frame, after the camera position was computed |
| `PerspectiveEvent` | no | the point of view changed to a different value (API 2) |
| `RenderCrosshairEvent` | yes | the vanilla crosshair is about to be drawn (API 2) |
| `RenderItemEvent` | yes | before the first-person swing transform is applied |
| `BlockOutlineEvent` | yes | before the targeted block outline is drawn |
| `SwingSpeedEvent` | no | the game asks how long a hand swing lasts |

### Render2DEvent

| Method | Type | Description |
|---|---|---|
| `render()` | [`Render`](../ui/render-2d.md) | the shared 2D drawing surface |
| `tickDelta()` | `float` | partial tick progress, 0..1 |
| `width()` | `float` | framebuffer width in physical pixels, not GUI-scaled |
| `height()` | `float` | framebuffer height in physical pixels, not GUI-scaled |

### Render3DEvent

| Method | Type | Description |
|---|---|---|
| `render()` | [`Render3D`](../ui/render-3d.md) | the shared world-space drawing surface |
| `tickDelta()` | `float` | partial tick progress, 0..1 |
| `camera()` | [`Vec`](../game/math.md) | camera world position in blocks, the origin world draws use |
| `viewMatrix()` | `float[]` | 16 floats, column-major, cloned per call (API 2) |
| `projectionMatrix()` | `float[]` | 16 floats, column-major, cloned per call (API 2) |

Both render events ignore `EventOptions` and are skipped when nothing is subscribed; `Render3DEvent` also needs a loaded world.

### CameraEvent

| Method | Type | Description |
|---|---|---|
| `yaw()` | `float` | camera yaw in degrees, interpolated by tick delta |
| `yaw(value)` | `void` | overrides camera yaw; applied to the camera |
| `pitch()` | `float` | camera pitch in degrees, interpolated by tick delta |
| `pitch(value)` | `void` | overrides camera pitch; applied to the camera |
| `x()` | `double` | camera X in world coordinates, interpolated |
| `x(value)` | `void` | overrides camera X; applied to the camera |
| `y()` | `double` | camera Y in world coordinates, eye offset included |
| `y(value)` | `void` | overrides camera Y; applied to the camera |
| `z()` | `double` | camera Z in world coordinates, interpolated |
| `z(value)` | `void` | overrides camera Z; applied to the camera |
| `position()` | [`Vec`](../game/math.md) | current x, y and z as a copy |
| `position(value)` | `void` | sets x, y and z from the vector |

Cancelling skips the rest of the camera update — third-person distance and clipping — but the values set here are still applied.

### PerspectiveEvent

| Method | Type | Description |
|---|---|---|
| `previous()` | [`Perspective`](../actions/control.md#game-settings) | perspective before the change, never null |
| `current()` | [`Perspective`](../actions/control.md#game-settings) | perspective after the change, never null |

Setting the same perspective again fires nothing.

### RenderItemEvent

| Method | Type | Description |
|---|---|---|
| `arm()` | [`Arm`](../game/inventory.md) | arm being rendered, LEFT or RIGHT |
| `swingProgress()` | `float` | swing animation progress, 0..1 |
| `translate(x, y, z)` | `void` | offsets the held-item matrix, item-space units (API 3) |
| `rotate(degrees, axisX, axisY, axisZ)` | `void` | rotates it about the axis, degrees (API 3) (no effect: axis length squared under 1e-6) |
| `rotateX(degrees)` | `void` | rotates it about `(1, 0, 0)`, degrees (API 3) |
| `rotateY(degrees)` | `void` | rotates it about `(0, 1, 0)`, degrees (API 3) |
| `rotateZ(degrees)` | `void` | rotates it about `(0, 0, 1)`, degrees (API 3) |
| `scale(x, y, z)` | `void` | scales it per axis (API 3) |

Cancelling leaves the matrix at the hand origin, so the swing transform is skipped for that hand.
The ops write into the live hand matrix and compose in call order, before the vanilla swing transform.

### Matrix

| Method | Type | Description |
|---|---|---|
| `translate(x, y, z)` | `void` | offsets the matrix, item-space units (API 3) |
| `rotate(degrees, axisX, axisY, axisZ)` | `void` | rotates it about the axis, degrees (API 3) (no effect: axis length squared under 1e-6) |
| `scale(x, y, z)` | `void` | scales it per axis (API 3) |

`RenderItemEvent.Matrix` is the adapter the event forwards to; `rotateX`, `rotateY` and `rotateZ` exist only on the event.

### BlockOutlineEvent

| Method | Type | Description |
|---|---|---|
| `x()` | `int` | outlined block X in world block coordinates |
| `y()` | `int` | outlined block Y in world block coordinates |
| `z()` | `int` | outlined block Z in world block coordinates |
| `box()` | [`Box`](../game/math.md) | outline shape offset to absolute world coordinates |

Fires only while an outline with a non-empty shape exists, and before both render events of the same frame.

### SwingSpeedEvent

| Method | Type | Description |
|---|---|---|
| `durationTicks()` | `int` | hand-swing animation length in ticks, vanilla base 6 |
| `durationTicks(value)` | `void` | overrides the duration; returned to the game, not clamped |

Local player only; swing progress divides by this value.

## Modules

| Event | Cancellable | Fires when |
|---|---|---|
| `ModuleToggleEvent` | no | a client module or a script toggled, after the state changed |

### ModuleToggleEvent

| Method | Type | Description |
|---|---|---|
| `name()` | `String` | module registry name, or the script toggle name |
| `enabled()` | `boolean` | state after the toggle |
| `fromScript()` | `boolean` | true when the toggled unit is a script |

## Everything else

| Event | Cancellable | Fires when |
|---|---|---|
| `ServerConnectEvent` | no | the client starts connecting to a multiplayer server |
| `ScreenCloseEvent` | no | any vanilla screen is closing |
| `FramebufferResizeEvent` | no | the game window resolution changed |

### ServerConnectEvent

| Method | Type | Description |
|---|---|---|
| `host()` | `String` | server hostname, empty when the address is null |
| `port()` | `int` | server port, 0 when the address is null |
| `address()` | `String` | host and port joined by a colon |

Fires before the socket is opened, so earlier than `WorldLoadEvent`.

### FramebufferResizeEvent

| Method | Type | Description |
|---|---|---|
| `width()` | `int` | framebuffer width in pixels, not GUI-scaled |
| `height()` | `int` | framebuffer height in pixels, not GUI-scaled |
