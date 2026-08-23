# How a script works

A loaded script is not a running script: the top level of the file runs at load, handlers and commands run only while the script is switched on. Everything the API offers hangs off the implicit receiver of the file, so every member on this page is callable unqualified.

```kotlin
name("Auto jump")
description("Jumps for you")
key(Key.G)

onEnable { chat.print("$id on") }
onDisable { chat.print("$id off") }
onUnload { storage.save() }

whenInGame { enabled = true }
```

## Loaded and switched on

| Method | Type | Description |
|---|---|---|
| `id` | `String` | script id = `.kts` file name without the extension |
| `enabled` | `Boolean` | on/off state, read and write; the write marshals to the client thread |
| `toggle()` | `Unit` | flips `enabled` |
| `onEnable(action)` | `Unit` | enable callback, several allowed, throws are logged |
| `onDisable(action)` | `Unit` | disable callback, several allowed, throws are logged |
| `onUnload(action)` | `Unit` | runs once on unload or reload (throws `ScriptStateException` when already unloaded) |
| `whenInGame(body)` | `Unit` | runs `body` only when player and world exist |

The top level runs once, at load. While the script is off its handlers, commands, hotkeys and scheduled tasks are unsubscribed; switching it back on re-arms them without re-running the file.

### Script

| Method | Type | Description |
|---|---|---|
| `Script.setEnabled(value)` | `void` | Java form of writing `enabled`, marshals to the client thread |
| `Script.defaultKey(key)` | `Script` | Java form of `key(key)`, returns the script |
| `Script.events()` | `Events` | the event registry behind `on<E> { }` — [Subscribing](../events/basics.md) |
| `Script.client()` | `Client` | the `client` root, table below |
| `Script.game()` | `Game` | the `game` root, table below |

`Script` is the Java object every member above forwards to; `id()`, `name()`, `description()`, `enabled()`, `toggle()`, `bind()`, `onEnable()`, `onDisable()` and `onUnload()` carry the names already on this page.
`Script` extends `SettingHost`, so every setting factory on [Kinds of settings](../settings/types.md) is a member of it too.

## Name, description, bind

| Method | Type | Description |
|---|---|---|
| `name(value)` | `Unit` | menu display name, trimmed (throws `ScriptException` when blank or over 32 chars) |
| `description(value)` | `Unit` | menu description, null becomes `""` |
| `key(key)` | `Unit` | default toggle key, written only while the bind is still empty |
| `bind` | `Bind` | the script's own toggle keybind — [Keys and binds](../actions/keys.md) |

## What you can reach

| Method | Type | Description |
|---|---|---|
| `client` | `Client` | client-side services, table below |
| `game` | `Game` | the running game session, table below |
| `user` | `User` | Nursultan account of the local user — [Your account](../extras/user.md) (API 2) |
| `chat` | `Chat` | chat printing and sending — [Messages](../ui/messages.md) |
| `text` | `Texts` | text component factory — [Styled text](../ui/text.md) |
| `log` | `Logger` | script console logger — [Messages](../ui/messages.md) |
| `storage` | `Config` | the script's default config — [Saving data](../settings/storage.md) |
| `configs` | `Configs` | this script's named configs — [Saving data](../settings/storage.md) |
| `assets` | `Assets` | read access to `scripts/assets` — [The assets folder](../extras/assets.md) (API 2) |
| `clipboard` | `Clipboard` | system clipboard — [Messages](../ui/messages.md) (API 2) |
| `filters` | `EntityFilters` | prebuilt entity predicates — [Entities and filters](../game/entities.md) |
| `keys` | `Keys` | keyboard and mouse state — [Keys and binds](../actions/keys.md) |
| `party` | `Party` | party info and the script message bus — [Party messages](../extras/party.md) (API 4) |
| `player` | `SelfPlayer` | the local player — [Your player](../game/player.md) |
| `world` | `World` | the loaded world — [World and blocks](../game/world.md) |
| `inventory` | `Inventory` | the player inventory — [Inventory and items](../game/inventory.md) |
| `container` | `Container` | the open screen handler — [Containers](../game/containers.md) |
| `recipes` | `Recipes` | the recipe book — [Containers](../game/containers.md) (API 2) |
| `inGame` | `Boolean` | true when both player and world exist |
| `interaction` | `Interaction` | attack, use, break — [Interaction](../actions/interaction.md) |
| `raycast` | `Raycast` | block and entity rays — [Rays and the crosshair](../game/raycast.md) |
| `control` | `Control` | movement flags and jump — [Movement](../actions/control.md) |
| `gameSettings` | `GameSettings` | Minecraft options — [Movement](../actions/control.md#game-settings) (API 2) |
| `slots` | `Slots` | hotbar slot selection — [Slots and armor](../actions/slots.md) |
| `armor` | `Armor` | worn armor comparison — [Slots and armor](../actions/slots.md) |
| `combat` | `Combat` | attack point and target marking — [Interaction](../actions/interaction.md) |
| `rotations` | `Rotations` | server-side yaw/pitch spoofing — [Rotations](../actions/rotations.md) |
| `prediction` | `Prediction` | movement and projectile prediction — [Prediction](../actions/prediction.md) |
| `gpu` | `Gpu` | mesh and pipeline registry — [Your own geometry](../ui/gpu.md) (API 2) |

`player`, `world`, `inventory`, `container`, `recipes`, `interaction`, `raycast` and `control` throw `ScriptStateException` outside a world; `inGame` is the guard.
Every root throws `ScriptStateException` after the script has been unloaded.

## The client object

| Method | Type | Description |
|---|---|---|
| `client.user()` | `User` | Nursultan account of the local user (API 2) |
| `client.chat()` | `Chat` | chat printing and sending, tagged with the script id |
| `client.text()` | `Texts` | text component factory |
| `client.log()` | `Logger` | script console logger, still usable after unload |
| `client.notifications()` | `Notifications` | client notification stack — [Messages](../ui/messages.md) |
| `client.tasks()` | `Tasks` | tick scheduler owned by this script — [Timers and tasks](../extras/tasks.md) |
| `client.storage()` | `Config` | the script's default config, file name `storage` |
| `client.configs()` | `Configs` | named per-script config store |
| `client.assets()` | `Assets` | read-only file access under `scripts/assets` (API 2) |
| `client.clipboard()` | `Clipboard` | system clipboard read and write (API 2) |
| `client.commands()` | `Commands` | `.`-prefixed command registration — [Your own commands](../extras/commands.md) |
| `client.modules()` | `Modules` | client module registry — [Client modules](../extras/modules.md) |
| `client.waypoints()` | `Waypoints` | client waypoint manager — [Waypoints](../extras/waypoints.md) |
| `client.party()` | `Party` | party info and the script message bus — [Party messages](../extras/party.md) (API 4) |
| `client.rotations()` | `Rotations` | spoofed server rotation handler |
| `client.combat()` | `Combat` | attack point, target marking |
| `client.slots()` | `Slots` | hotbar and held-slot control |
| `client.armor()` | `Armor` | worn armor accessor |
| `client.prediction()` | `Prediction` | movement and projectile prediction |
| `client.timer()` | `Timer` | a new independent stopwatch on every call — [Timers and tasks](../extras/tasks.md) |
| `client.filters()` | `EntityFilters` | entity predicate factory, still usable after unload |
| `client.keys()` | `Keys` | keyboard, mouse and cursor state |
| `client.fonts()` | `Fonts` | per-session font registry — [2D render](../ui/render-2d.md) |
| `client.shaders()` | `Shaders` | per-script shader registry — [Shaders](../ui/shaders.md) |
| `client.gpu()` | `Gpu` | per-script GPU buffer and pipeline registry (API 2) |
| `client.textures()` | `Textures` | per-script texture registry — [2D render](../ui/render-2d.md) |

`client.fps()`, `tick()`, `millis()`, `nanos()`, `tickDelta()` and `onClientThread()` are documented on [Timers and tasks](../extras/tasks.md).

## The game object

| Method | Type | Description |
|---|---|---|
| `game.inGame()` | `boolean` | true when both player and world exist |
| `game.connected()` | `boolean` | true while a play network handler exists |
| `game.player()` | `SelfPlayer` | the local player — [Your player](../game/player.md) |
| `game.world()` | `World` | the client world — [World and blocks](../game/world.md) |
| `game.inventory()` | `Inventory` | the player inventory — [Inventory and items](../game/inventory.md) |
| `game.control()` | `Control` | movement-state control — [Movement](../actions/control.md) |
| `game.interaction()` | `Interaction` | attack, use, break — [Interaction](../actions/interaction.md) |
| `game.raycast()` | `Raycast` | block and entity ray casting — [Rays and the crosshair](../game/raycast.md) |
| `game.server()` | `Server` | current server info — [Server, scoreboard, tab list](../game/server.md) (throws `ScriptStateException` when not connected) |
| `game.packets()` | `Packets` | outgoing packet sender, never throws — [Packets](../actions/packets.md) |
| `game.screenOpen()` | `boolean` | true when any screen is open |
| `game.screenKind()` | `ScreenKind` | kind of the open screen, `NONE` when there is none |
| `game.screen()` | `Screen?` | snapshot of the open screen, null when none — [Server, scoreboard, tab list](../game/server.md) |
| `game.container()` | `Container` | the open screen handler — [Containers](../game/containers.md) |
| `game.recipes()` | `Recipes` | the recipe book — [Containers](../game/containers.md) (API 2) |
| `game.settings()` | `GameSettings` | game options, never throws — [Movement](../actions/control.md#game-settings) (API 2) |
| `game.translate(key, arguments)` | `String` | translated Minecraft text, `""` for a blank key |
| `game.hasTranslation(key)` | `boolean` | true when the key exists in the loaded language |
| `game.language()` | `String` | language code, e.g. `en_us` |
| `game.item(itemId)` | `Item` | new stack of size 1, namespace defaults to `minecraft:` (throws `ScriptException` when the id is unparseable) |
| `game.closeScreen()` | `void` | closes the open screen on the client thread |
| `game.disconnect()` | `void` | leaves the server on the client thread |

`player()`, `world()`, `inventory()`, `control()`, `interaction()`, `raycast()`, `container()` and `recipes()` throw `ScriptStateException` outside a world.

## Version requirement

| Method | Type | Description |
|---|---|---|
| `requireApi(minimum)` | `Unit` | refuses the load on an older client (throws `ScriptApiException` when `ApiVersion.CURRENT` < `minimum`) |

This client provides API version 5 — see [API versions](../extras/api-versions.md).
