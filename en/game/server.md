# Server, scoreboard, tab list

## Where am I connected

```kotlin
game.connected()             // there is a connection
game.inGame()                // the world is loaded and the player exists

val server = game.server()

server.name()
server.address()             // "mc.example.com:25565"
server.type()                // SINGLEPLAYER, LAN, REALM, MULTIPLAYER
server.protocolVersion()
server.pingMs()
server.tps()                 // how many ticks per second the server manages
```

`tps()` is measured from how often ticks arrive — useful for not hammering the server with actions while it lags.

You can disconnect from code:

```kotlin
if (player.health() < 4f) {
    game.disconnect()
}
```

## The side scoreboard

```kotlin
val board = game.server().scoreboard()

board.visible()
board.title()                // the header
board.lines()                // lines top to bottom
```

Often you only want to know which mode you are in:

```kotlin
if (board.contains("BedWars", "Bed Wars")) {
    // we are on bedwars
}
```

`contains` checks the title and the lines for any of the words you pass and does not care about case.

### Objectives and scores

`lines()` is the sidebar flattened into strings. When you need the numbers, or an objective that is not on the sidebar, go through the objectives:

```kotlin
val board = game.server().scoreboard()

board.objectives()               // every objective the server sent
board.objective("kills")         // one by its internal name
board.display("sidebar")         // whichever one is in a display slot
board.holders()                  // every name that has a score anywhere
```

```kotlin
val kills = board.objective("kills") ?: return
kills.displayName().string()
kills.criterion()                // "dummy", "health", "playerKillCount"...
kills.renderType()               // "integer" or "hearts"
kills.score(player.name())

for (entry in kills.entries()) {
    entry.owner()                // whose score it is
    entry.value()
    entry.hidden()
    entry.name()                 // Text, as drawn
}
```

### Teams

```kotlin
board.teams()
board.team("red")
board.teamOf(player.name())      // which team a name is on
```

```kotlin
val team = board.teamOf(target.name()) ?: return

team.displayName().string()
team.prefix()                    // Text
team.suffix()
team.color()                     // "red", "aqua"...
team.colorRgb()                  // 0xAARRGGBB, or -1 when the team has no colour
team.members()
team.friendlyFire()
team.nametagVisibility()         // "always", "never", "hide_for_other_teams"...
team.collisionRule()
team.contains(player.name())
```

The whole scoreboard is read-only: writing to it would only desync your own view, the server would never hear about it.

## Boss bars

```kotlin
for (bar in game.server().bossBars()) {
    bar.name()                   // Text
    bar.percent()                // 0..1
    bar.color()                  // "pink", "blue", "red", "green", "yellow", "purple", "white"
    bar.style()                  // "progress", "notched_6", "notched_10", "notched_12", "notched_20"
    bar.uuid()
}
```

Servers use boss bars for round timers, event countdowns and region names, so this is often the quickest way to read game state:

```kotlin
val timer = game.server().bossBars()
    .firstOrNull { it.name().string().contains("Round") }
```

## Translations

```kotlin
game.translate("block.minecraft.stone")           // "Stone", in the player's language
game.translate("chat.type.text", "Notch", "hi")   // fills the %s slots
game.hasTranslation("mymod.key")
game.language()                                    // "en_us"
```

Handy for comparing against what the player actually sees, instead of hardcoding English.

You can also build an item out of an id, without having one in your inventory:

```kotlin
val apple = game.item("golden_apple")
apple.name()
apple.nutrition()
```

## The tab list

```kotlin
val tab = game.server().tabList()

tab.header()
tab.footer()
tab.players()                // the roster, in the order the tab shows it

tab.headerContains("anarchy")
tab.footerContains("season")
```

`players()` is the roster the server sent, not the players loaded around you: everyone is in it, including the ones on the far side of the map, in another world, or hidden from the list entirely. Each row is a `TabEntry`:

```kotlin
for (row in tab.players()) {
    row.name()               // profile name
    row.uuid()
    row.displayName()        // what the tab draws, as plain text
    row.pingMs()
    row.gameMode()
    row.listed()             // false for rows the tab does not draw
    row.skinTexture()        // the skin, or null
    row.player()             // the entity, or null when it is not loaded
}
```

A row is not an entity. Position, health and everything else about the body live on `player()` — a [player entity](entities.md) — and that is `null` whenever the player is out of render distance, which on a big server is most of the roster:

```kotlin
val nearby = tab.players().mapNotNull { it.player() }
```

Filter by `listed()` when you want only what the tab actually draws — servers turn it off to keep NPCs and staff out of the list while still sending their skins.

`skinTexture()` works for the whole roster, entity or no entity, so a tab list with heads down the side needs nothing loaded — see [drawing a piece of a texture](../ui/render-2d.md).

## Screens

```kotlin
game.screenOpen()            // is any screen open
game.screenKind()            // NONE, INVENTORY, CREATIVE, CONTAINER, CHAT, OTHER
game.closeScreen()
```

Handy for staying out of the way while the player digs through a chest:

```kotlin
if (game.screenKind() == ScreenKind.CONTAINER) return@on
```

For the details — the title above all — ask for the screen itself. It is `null` when nothing is open:

```kotlin
val screen = game.screen() ?: return

screen.kind()
screen.title()               // Text: what the server called this menu
screen.handled()             // true for anything with slots in it
screen.syncId()              // the id the server gave this menu
screen.size()                // how many slots it has
```

Servers name their menus, so the title is how you tell one from another:

```kotlin
if (game.screen()?.title()?.string() == "Shop") {
    // we are looking at the shop
}
```

There is a screen-closed event too — `ScreenCloseEvent`.

## Detect the server once

There is no point checking the scoreboard every tick. Once per world is enough:

```kotlin
var bedwars = false

on<WorldLoadEvent> {
    afterTicks(40) {
        bedwars = game.server().scoreboard().contains("BedWars")
    }
}
```

The delay is there because the scoreboard does not arrive the instant the world loads.
