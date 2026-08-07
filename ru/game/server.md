# Сервер, табло, таблист

## Куда я подключён

```kotlin
game.connected()             // есть подключение
game.inGame()                // мир загружен и игрок есть

val server = game.server()

server.name()
server.address()             // "mc.example.com:25565"
server.type()                // SINGLEPLAYER, LAN, REALM, MULTIPLAYER
server.protocolVersion()
server.pingMs()
server.tps()                 // сколько тиков в секунду тянет сервер
```

`tps()` считается по тому, как часто приходят тики — полезно, чтобы не молотить действиями, когда сервер лагает.

Отключиться можно из кода:

```kotlin
if (player.health() < 4f) {
    game.disconnect()
}
```

## Табло сбоку

```kotlin
val board = game.server().scoreboard()

board.visible()
board.title()                // заголовок
board.lines()                // строки сверху вниз
```

Часто нужно просто понять, на каком ты режиме:

```kotlin
if (board.contains("BedWars", "Бедвары")) {
    // мы на бедварах
}
```

`contains` проверяет заголовок и строки на любое из переданных слов и не мучает регистром.

### Цели и очки

`lines()` — это сайдбар, сплющенный в строки. Когда нужны числа или цель, которой на сайдбаре нет, иди через цели:

```kotlin
val board = game.server().scoreboard()

board.objectives()               // все цели, которые прислал сервер
board.objective("kills")         // одна по внутреннему имени
board.display("sidebar")         // та, что стоит в слоте отображения
board.holders()                  // все имена, у которых где-то есть очки
```

```kotlin
val kills = board.objective("kills") ?: return
kills.displayName().string()
kills.criterion()                // "dummy", "health", "playerKillCount"...
kills.renderType()               // "integer" или "hearts"
kills.score(player.name())

for (entry in kills.entries()) {
    entry.owner()                // чьи это очки
    entry.value()
    entry.hidden()
    entry.name()                 // Text, как нарисовано
}
```

### Команды

```kotlin
board.teams()
board.team("red")
board.teamOf(player.name())      // в какой команде это имя
```

```kotlin
val team = board.teamOf(target.name()) ?: return

team.displayName().string()
team.prefix()                    // Text
team.suffix()
team.color()                     // "red", "aqua"...
team.colorRgb()                  // 0xAARRGGBB, или -1, если у команды нет цвета
team.members()
team.friendlyFire()
team.nametagVisibility()         // "always", "never", "hide_for_other_teams"...
team.collisionRule()
team.contains(player.name())
```

Табло целиком только на чтение: запись рассинхронизировала бы твою картинку, а сервер о ней всё равно не узнал бы.

## Боссбары

```kotlin
for (bar in game.server().bossBars()) {
    bar.name()                   // Text
    bar.percent()                // 0..1
    bar.color()                  // "pink", "blue", "red", "green", "yellow", "purple", "white"
    bar.style()                  // "progress", "notched_6", "notched_10", "notched_12", "notched_20"
    bar.uuid()
}
```

Серверы вешают на боссбары таймеры раундов, обратный отсчёт ивентов и названия регионов, так что это часто самый быстрый способ прочитать состояние игры:

```kotlin
val timer = game.server().bossBars()
    .firstOrNull { it.name().string().contains("Раунд") }
```

## Переводы

```kotlin
game.translate("block.minecraft.stone")           // «Камень», на языке игрока
game.translate("chat.type.text", "Notch", "hi")   // подставит в %s
game.hasTranslation("mymod.key")
game.language()                                    // "ru_ru"
```

Пригодится, чтобы сравнивать с тем, что игрок реально видит, а не зашивать английский.

Ещё можно собрать предмет по идентификатору, не имея его в инвентаре:

```kotlin
val apple = game.item("golden_apple")
apple.name()
apple.nutrition()
```

## Таблист

```kotlin
val tab = game.server().tabList()

tab.header()
tab.footer()
tab.players()                // список, в том порядке, в каком его показывает таб

tab.headerContains("anarchy")
tab.footerContains("сезон")
```

`players()` — это список, который прислал сервер, а не игроки, прогруженные вокруг тебя: в нём все, включая тех, кто на другом конце карты, в другом мире или вообще спрятан из таба. Каждая строка — `TabEntry`:

```kotlin
for (row in tab.players()) {
    row.name()               // имя профиля
    row.uuid()
    row.displayName()        // то, что рисует таб, обычным текстом
    row.pingMs()
    row.gameMode()
    row.listed()             // false для строк, которые таб не рисует
    row.skinTexture()        // скин или null
    row.player()             // сущность или null, если игрок не прогружен
}
```

Строка — это не сущность. Позиция, здоровье и всё остальное про тело живёт на `player()` — это [сущность игрока](entities.md) — и там `null`, когда игрок вне прогрузки, а на большом сервере это почти весь список:

```kotlin
val nearby = tab.players().mapNotNull { it.player() }
```

Фильтруй по `listed()`, когда нужно только то, что таб реально рисует: сервера гасят его, чтобы держать NPC и админов вне списка, но всё равно присылать их скины.

`skinTexture()` работает на весь список, есть сущность или нет, так что таблисту с головами сбоку ничего прогружать не нужно — см. [кусок текстуры](../ui/render-2d.md).

## Экраны

```kotlin
game.screenOpen()            // открыт ли какой-нибудь экран
game.screenKind()            // NONE, INVENTORY, CREATIVE, CONTAINER, CHAT, OTHER
game.closeScreen()
```

Пригодится, чтобы не действовать, пока игрок копается в сундуке:

```kotlin
if (game.screenKind() == ScreenKind.CONTAINER) return@on
```

За подробностями — прежде всего за заголовком — спрашивай сам экран. Он `null`, когда ничего не открыто:

```kotlin
val screen = game.screen() ?: return

screen.kind()
screen.title()               // Text: как сервер назвал это меню
screen.handled()             // true у всего, где есть слоты
screen.syncId()              // идентификатор, который сервер выдал меню
screen.size()                // сколько в нём слотов
```

Серверы дают своим меню имена, так что заголовок — это способ отличить одно от другого:

```kotlin
if (game.screen()?.title()?.string() == "Магазин") {
    // мы смотрим в магазин
}
```

Есть и событие закрытия экрана — `ScreenCloseEvent`.

## Определить сервер один раз

Проверять табло каждый тик незачем. Достаточно один раз при заходе в мир:

```kotlin
var bedwars = false

on<WorldLoadEvent> {
    afterTicks(40) {
        bedwars = game.server().scoreboard().contains("BedWars")
    }
}
```

Задержка нужна потому, что табло приходит не сразу после загрузки мира.
