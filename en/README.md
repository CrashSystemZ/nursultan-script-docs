# What this is

Nursultan scripts are plain Kotlin files with a `.kts` extension that the client compiles while the game is running. Save the file and the script reloads — no restart needed.

One file is one script. Here is a whole working one:

```kotlin
name("Hello")
description("My first script")

chat.print("hi")

on<ClientTickEvent> {
    // runs every game tick while the script is on
}
```

## What you can build

* **Your own module** — settings, a bind, a switch in the Scripts tab. See [How a script works](start/lifecycle.md) and [Kinds of settings](settings/types.md).
* **Reactions to the game** — ticks, damage, attacks, input, packets, entities spawning and so on. See [Events](events/basics.md).
* **Reading the world** — player, blocks, entities, inventory, server and so on. See [Game data](game/player.md).
* **Actions** — attack, place, break, turn your head, swap slots, send packets and so on. See [Actions](actions/interaction.md).
* **Your own graphics** — HUD, boxes and lines in the world, custom shaders and so on. See [Interface](ui/render-2d.md).
* **Your own commands** — with Tab completion, just like the client's own and so on. See [Your own commands](extras/commands.md).

## Where scripts live

`%APPDATA%\Nursultan\scripts`

That folder holds **only** finished `.kts` files. Writing scripts is nicer in a separate project folder — see [The project in your IDE](start/ide.md).

The file name is the script id: `auto-jump.kts` shows up in the menu as `auto-jump` until you give it a `name(...)`.

## The Scripts tab

Everything is managed from the **Scripts** tab in the client menu.

* While compiling, the script's card is inactive. An error turns it red and shows the message.
* Every script gets one card: the **switch** turns it on and off, **the dots or right click** opens settings, **the keyboard icon or middle click** sets a bind.
* State survives restarts: on/off, bind and settings all come back.
