# Server, scoreboard, tab list

`game.server()` describes the connection you are on and exposes the scoreboard, the tab list and the boss bars. It throws `ScriptStateException` when the client is not connected.

```kotlin
on<ClientTickEvent> {
    val board = game.server().scoreboard()
    if (!board.visible()) return@on
    log.info(board.title())
    board.lines().forEach { log.info(it) }

    val me = game.server().tabList().players().firstOrNull { it.uuid() == player.uuid() }
    log.info("ping ${me?.pingMs() ?: -1} ms")
}
```

## The connection

| Method | Type | Description |
|---|---|---|
| `server.name()` | `String` | server-list entry name, `"singleplayer"` without an entry |
| `server.address()` | `String` | server-list address `host:port`, `"singleplayer"` without an entry |
| `server.type()` | `ServerType` | kind of connection, MULTIPLAYER when unknown |
| `server.protocolVersion()` | `int` | protocol version from the server-list entry, 0 without one |
| `server.pingMs()` | `int` | your own tab-list latency in ms, 0 when unavailable |
| `server.tps()` | `float` | measured server tick rate, rounded to 0.1, 20.0 out of world |
| `server.scoreboard()` | `Scoreboard` | the client scoreboard view |
| `server.tabList()` | `TabList` | the player list view |
| `server.bossBars()` | `List<BossBar>` | boss bars on the HUD, empty without an in-game HUD |

| Constant | Description |
|---|---|
| `SINGLEPLAYER` | integrated server, no server-list entry |
| `LAN` | LAN-discovered server |
| `REALM` | Minecraft Realm |
| `MULTIPLAYER` | any other remote server, also the unknown fallback |

## The sidebar

| Method | Type | Description |
|---|---|---|
| `scoreboard.visible()` | `boolean` | true when a sidebar objective is resolved |
| `scoreboard.title()` | `String` | plain text of the sidebar title, `""` when none |
| `scoreboard.lines()` | `List<String>` | plain text of non-hidden sidebar entries, scoreboard order |
| `scoreboard.contains(vararg needles)` | `boolean` | case-insensitive substring over the title only, never the lines |

The sidebar objective is the team-colour slot matching your own team colour, falling back to the plain `sidebar` slot.

## Objectives and scores

| Method | Type | Description |
|---|---|---|
| `scoreboard.objectives()` | `List<Objective>` | every objective known to the client, empty out of world |
| `scoreboard.objective(name)` | `Objective?` | objective by internal name, null when absent |
| `scoreboard.display(slot)` | `Objective?` | objective in a display slot: `list`, `sidebar`, `below_name`, `sidebar.team.red` |
| `scoreboard.holders()` | `List<String>` | every known score-holder name, empty out of world |

| Method | Type | Description |
|---|---|---|
| `objective.name()` | `String` | internal objective name |
| `objective.displayName()` | `Text` | styled display name |
| `objective.criterion()` | `String` | criterion name, e.g. `dummy`, `health`, `playerKillCount` |
| `objective.renderType()` | `String` | `integer` or `hearts` |
| `objective.slots()` | `List<String>` | display slots this objective occupies, empty out of world |
| `objective.score(holder)` | `int` | that holder's score, 0 when there is no entry |
| `objective.entries()` | `List<ScoreEntry>` | every score entry of this objective |

| Method | Type | Description |
|---|---|---|
| `entry.owner()` | `String` | score-holder name |
| `entry.value()` | `int` | the numeric score |
| `entry.hidden()` | `boolean` | true when the server marked the entry as not displayed |
| `entry.name()` | `Text` | styled name as drawn on the sidebar |
| `entry.formattedValue()` | `Text` | styled score value in the entry's own number format |

`Text` is documented on [Styled text](../ui/text.md).

## Teams

| Method | Type | Description |
|---|---|---|
| `scoreboard.teams()` | `List<Team>` | every team, empty out of world |
| `scoreboard.team(name)` | `Team?` | team by internal name, null when absent |
| `scoreboard.teamOf(holder)` | `Team?` | team that score-holder name belongs to |

| Method | Type | Description |
|---|---|---|
| `team.name()` | `String` | internal team name |
| `team.displayName()` | `Text` | styled team display name |
| `team.prefix()` | `Text` | styled prefix prepended to member names |
| `team.suffix()` | `Text` | styled suffix appended to member names |
| `team.color()` | `String` | formatting colour name, e.g. `red`, `aqua` |
| `team.colorRgb()` | `int` | ARGB with 0xFF alpha, -1 when the colour has no RGB |
| `team.members()` | `List<String>` | member score-holder names, immutable copy |
| `team.friendlyFire()` | `boolean` | vanilla friendly-fire flag |
| `team.showFriendlyInvisibles()` | `boolean` | vanilla see-friendly-invisibles flag |
| `team.nametagVisibility()` | `String` | `always`, `never`, `hide_for_other_teams`, `hide_for_own_team` |
| `team.deathMessageVisibility()` | `String` | same four values, applied to death messages |
| `team.collisionRule()` | `String` | `always`, `never`, `push_other_teams`, `push_own_team` |
| `team.decorate(value)` | `Text` | prefix + value + suffix with team formatting, null value is `""` |
| `team.contains(holder)` | `boolean` | membership test, false for null |

The scoreboard is read-only; nothing written into these objects reaches the server.

## Boss bars

| Method | Type | Description |
|---|---|---|
| `bar.uuid()` | `String` | boss bar UUID string |
| `bar.name()` | `Text` | styled boss bar title |
| `bar.percent()` | `float` | fill fraction, 0..1 |
| `bar.color()` | `String` | `pink`, `blue`, `red`, `green`, `yellow`, `purple`, `white` |
| `bar.style()` | `String` | `progress`, `notched_6`, `notched_10`, `notched_12`, `notched_20` |

## The tab list

| Method | Type | Description |
|---|---|---|
| `tab.header()` | `String` | plain text of the tab header, `""` when absent |
| `tab.footer()` | `String` | plain text of the tab footer, `""` when absent |
| `tab.headerContains(vararg needles)` | `boolean` | case-insensitive substring over the trimmed header |
| `tab.footerContains(vararg needles)` | `boolean` | the same test against the footer |
| `tab.players()` | `List<TabEntry>` | whole roster in vanilla tab order, empty when not connected |

| Method | Type | Description |
|---|---|---|
| `row.name()` | `String` | profile name |
| `row.uuid()` | `String` | profile UUID string |
| `row.displayName()` | `String` | plain text of the tab name, falls back to `name()` |
| `row.pingMs()` | `int` | latency of that entry in milliseconds |
| `row.pingMs(value)` | `void` | client-side latency write, the next server update overwrites it (API 2) |
| `row.gameMode()` | `GameMode` | SURVIVAL, CREATIVE, ADVENTURE, SPECTATOR; SURVIVAL when unset |
| `row.listed()` | `boolean` | true when the entry is one the tab draws |
| `row.skinTexture()` | `Texture?` | skin body texture, null without skins or in StreamerMode |
| `row.player()` | `PlayerEntity?` | loaded entity with that UUID, null when out of world or not loaded |

A row is a profile entry, not an entity: the roster holds every player the server sent, loaded or not.
`Texture` is documented on [2D render](../ui/render-2d.md), `PlayerEntity` and `GameMode` on [Entities and filters](entities.md).

## Screens

| Method | Type | Description |
|---|---|---|
| `screen.kind()` | `ScreenKind` | classification of the open screen |
| `screen.title()` | `Text` | styled screen title |
| `screen.handled()` | `boolean` | true when the screen has a screen handler |
| `screen.syncId()` | `int` | screen handler sync id, 0 when not handled |
| `screen.size()` | `int` | screen handler slot count, 0 when not handled |

| Constant | Description |
|---|---|
| `NONE` | no screen open |
| `INVENTORY` | vanilla survival inventory screen |
| `CREATIVE` | creative inventory screen |
| `CONTAINER` | any other handled screen: chest, villager, shop menu |
| `CHAT` | the chat input screen |
| `OTHER` | any other screen: menus, options |

`game.screen()` returns null when nothing is open; `game.screenOpen()`, `game.screenKind()` and `game.closeScreen()` are on [How a script works](../start/lifecycle.md). The closing of a screen fires `ScreenCloseEvent`.

## Translations

`game.translate(key, args)`, `game.hasTranslation(key)` and `game.language()` are on [How a script works](../start/lifecycle.md).
