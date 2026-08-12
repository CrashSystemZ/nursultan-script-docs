# Event list

Cancellable events are marked in the "Cancel" column. How to subscribe and what priority means is in [Subscribing](basics.md).

## Ticks

| Event | When | Cancel |
|---|---|---|
| `ClientTickEvent` | every client tick, even without a world | |
| `PrePlayerTickEvent` | before the player ticks | |
| `PlayerTickEvent` | inside the player tick, before movement is worked out | |
| `PostPlayerTickEvent` | the player tick is over — movement is done and the packets are already sent | |

`ClientTickEvent` always fires, so it almost always needs an `if (!inGame) return@on`. `PrePlayerTickEvent` is where you get in before the player works out its movement for the tick. `PostPlayerTickEvent` is the opposite end: use it to read where the tick actually left you.

## Movement

| Event | When | Cancel |
|---|---|---|
| `MovePacketEvent` | right before the position and rotation packets for the tick go out | yes |
| `MoveEvent` | before the player's move is applied, `movement()` can be rewritten | yes |
| `TravelEvent` | before vanilla movement physics for the tick | yes |
| `MoveRelativeEvent` | after movement input was turned into velocity, read only | |
| `EntityPushEvent` | an entity pushes you or you push it | yes |
| `BlockPushEvent` | the game pushes you out of a block you are stuck in | yes |
| `WebCollisionEvent` | you touch a cobweb, `x()`/`y()`/`z()` of the web | yes |
| `TimerEvent` | every frame, when the game decides how many ticks to run | |

`MovePacketEvent` is the last point where the position the server sees is still yours to change. Every field reads and writes: `x()`, `y()`, `z()`, `yaw()`, `pitch()`, `onGround()`, `sprinting()`. Cancelling it sends nothing at all that tick — the server keeps your last known position.

```kotlin
on<MovePacketEvent> {
    it.y(it.y() - 0.05)
    it.onGround(true)
}
```

`MoveEvent` is one step lower: `movement()` is the delta the player is about to move by, after input and physics, before collisions. Rewrite it to change where you actually go.

```kotlin
on<MoveEvent> {
    it.set(it.x() * 1.4, it.y(), it.z() * 1.4)
}
```

`TravelEvent` is lower still — cancel it and vanilla physics does not run for that tick at all: no gravity, no drag, no input. Nothing moves you unless you move yourself.

`MoveRelativeEvent` only reports: `speed()` is the acceleration the game used and `input()` is the raw movement input. Writing to it changes nothing, the game has already applied the value.

`TimerEvent` carries `timer()` — the game speed multiplier for the frame, `1f` is normal. It is reset to `1f` every frame, so set it every time you want it to hold.

```kotlin
on<TimerEvent> { it.timer(1.6f) }
```

Nothing clamps that number: `0f` stops time and divides by zero downstream, negatives run it backwards. Keep it sane. The client's own Timer module writes the same value, so whichever of you runs last wins — use `Priority.LAST` if you need it to be you.

## Combat

| Event | When | Cancel |
|---|---|---|
| `AttackEvent` | you hit an entity, `target()` | yes |
| `AttackedEvent` | the hit went through, `target()` | |
| `UseItemEvent` | item use starts, `hand()` | yes |
| `StopUsingItemEvent` | the client is about to release right click — cancel it to keep eating/drinking/charging | yes |
| `FinishUsingItemEvent` | an item finished being used — food eaten, potion drunk, `item()` | |
| `RightClickEvent` | the client is about to process a right click, `hand()` | yes |
| `JumpEvent` | the player jumped | |
| `SlowdownEvent` | the game slows movement down (food, shield, bow); `multiplier()` can be rewritten | |
| `TargetUpdateEvent` | the client's combat target changed | |

`TargetUpdateEvent` carries `previous()` and `target()` — both can be null, and `lost()` is the short way to ask whether the target is gone. It is the *client's* target, published by whatever combat module is running (AttackAura, TriggerBot, AimAssist) — with none of them on, it never fires.

## Blocks

| Event | When | Cancel |
|---|---|---|
| `BlockHitEvent` | the client starts or continues breaking a block, `x()`/`y()`/`z()`, `side()` | yes |
| `BlockBreakingEvent` | once a tick, before the client advances block breaking | yes |
| `BlockPlacedEvent` | you placed a block, `block()` | |

`BlockBreakingEvent` fires every tick you are in a world, not only while you are digging — `breaking()` tells you which it is. Cancel it and the client skips its breaking step for that tick, including the bookkeeping that stops a break you have walked away from, so cancel it on the ticks you mean and not blindly.

`BlockPlacedEvent` is your own placements only: it comes from the client's predicted place, so blocks other players put down never reach it — those arrive as plain block updates. It gives you a full [`Block`](../game/world.md), so `id()`, `position()` and everything else is on it directly.

## Inventory

| Event | When | Cancel |
|---|---|---|
| `SlotClickEvent` | a slot in an open container was clicked | yes |
| `EffectEvent` | a status effect on your player was added, refreshed or removed | |

`SlotClickEvent` carries `syncId()`, `slotId()`, `button()`, `action()` and `item()` — the stack that is in the slot right now. Cancel it and the click never reaches the server.

```kotlin
on<SlotClickEvent> {
    if (it.action() == SlotAction.THROW) it.cancel()
}
```

`EffectEvent` carries `effect()` and `action()`. `action()` is `ADD`, `UPDATE`, `REMOVE` or `PLAYER_INIT`; on `PLAYER_INIT` the player object is only being created and `effect()` is `null`.

## Input

| Event | When | Cancel |
|---|---|---|
| `KeyEvent` | a key or mouse button went down or up | yes |
| `CharEvent` | a character was typed | yes |
| `MouseScrollEvent` | the mouse wheel | yes |
| `MoveInputEvent` | movement input for the tick, fields can be rewritten | yes |
| `LookInputEvent` | mouse movement, `deltaX`/`deltaY` can be rewritten | yes |

`KeyEvent` carries `key()`, `mods()`, `action()`, plus the short `pressed()`, `released()`, `mouse()` and `matches(Key.G)`.

`MoveInputEvent` is what the game is about to do: `forward()`, `backward()`, `left()`, `right()`, `jump()`, `sneak()`, `sprint()`. Every field both reads and writes:

```kotlin
on<MoveInputEvent>(priority = Priority.LAST) {
    it.sprint(true)
    it.jump(true)
}
```

## World and entities

| Event | When | Cancel |
|---|---|---|
| `EntitySpawnEvent` | an entity appeared, `entity()` | |
| `EntityRemoveEvent` | an entity went away, `entity()` | |
| `WorldLoadEvent` | a world loaded — joining a server, changing dimension | |
| `ScreenCloseEvent` | a screen closed (inventory, chest, menu) | |
| `PlaySoundEvent` | the world is about to play a sound | yes |
| `PotionImpactEvent` | a thrown potion hit something | |
| `ServerConnectEvent` | you started connecting to a server | |

`WorldLoadEvent` is a good place to reset anything tied to one world.

`PlaySoundEvent` gives you `id()` and `category()` to read, and `volume()`, `pitch()`, `x()`, `y()`, `z()` to read and rewrite. Cancel it for silence.

```kotlin
on<PlaySoundEvent> {
    if (it.id() == "minecraft:entity.player.levelup") it.cancel()
}
```

`PotionImpactEvent` fires once per potion and carries `potion()`, `position()` and `hitEntity()` — the last one is `null` when the potion hit a block instead.

`ServerConnectEvent` carries `host()`, `port()` and `address()`, and fires before the world loads — earlier than `WorldLoadEvent`.

## Packets

| Event | When | Cancel |
|---|---|---|
| `PacketReceiveEvent` | a packet arrived from the server, `packet()` | yes |
| `PacketSendEvent` | the client is about to send a packet, `packet()` | yes |

These fire **on the network thread**. Details and the packet list are in [Packets](../actions/packets.md).

## Rendering

| Event | When | Cancel |
|---|---|---|
| `Render2DEvent` | every frame, the client overlay layer | |
| `Render3DEvent` | every frame, in the world | |
| `BlockOutlineEvent` | the game is about to draw the outline of the block you are looking at | yes |
| `CameraEvent` | the camera position and rotation for the frame, all writable | yes |
| `SwingSpeedEvent` | the game asks how long a hand swing lasts | |
| `RenderItemEvent` | before the swing transform of the first-person item is applied | yes |
| `FramebufferResizeEvent` | the window changed size | |
| `PerspectiveEvent` | the point of view changed — F5 or a script | |

`Render2DEvent` carries `render()`, the screen size `width()` × `height()` and `tickDelta()`. `Render3DEvent` carries `render()`, `tickDelta()` and the camera position `camera()`. See [2D render](../ui/render-2d.md) and [3D render](../ui/render-3d.md).

`BlockOutlineEvent` carries the block coordinates `x()`, `y()`, `z()` and `box()` — the outline shape in world coordinates, so a slab gives you a half-height box. Cancel it and the vanilla outline is gone; draw your own in `Render3DEvent`:

```kotlin
var target: Box? = null

on<BlockOutlineEvent> { e ->
    e.cancel()
    target = e.box()
}

on<Render3DEvent> { e ->
    e.render().box(target ?: return@on, Colors.CYAN, false)
}
```

It fires during the world render, before both render events of the same frame, so what you cache is always current. It stops firing the moment you look away — clear your copy in `ClientTickEvent` if nothing arrived.

`CameraEvent` gives you `yaw()`, `pitch()`, `x()`, `y()`, `z()` — all of them read and write. Cancelling skips the rest of the camera update — third-person distance and the sleeping tilt — but keeps the values you set.

`SwingSpeedEvent` carries `durationTicks()`: how many ticks one swing takes, `6` by default. Lower it for a faster animation, but not to `0` — swing progress is a division by it.

`RenderItemEvent` carries `arm()` and `swingProgress()`. Cancel it and the swing transform is not applied — the item stays still in your hand.

`FramebufferResizeEvent` carries `width()` and `height()` in real pixels — the same space `Render2DEvent` draws in. Use it to drop anything you sized to the old window.

`PerspectiveEvent` carries `previous()` and `current()`, both a `Perspective`. It is the option the player switched, not what a client module forces on the frame — see [Game settings](../actions/control.md#game-settings).

## Modules

| Event | When | Cancel |
|---|---|---|
| `ModuleToggleEvent` | a client module or a script was switched | |

Carries `name()`, `enabled()` and `fromScript()` — the last one tells you whether it was a script or a built-in module.

## What tickDelta is

The game thinks 20 times per second but draws far more often. `tickDelta` is a fraction from 0 to 1 telling you how far the frame is past the last tick. You use it so movement on screen is smooth instead of stepping 20 times a second.
