# Сервер, табло, таблист

`game.server()` описывает подключение и даёт доступ к табло, таблисту и боссбарам. Кидает `ScriptStateException`, когда клиент не подключён.

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

## Подключение

| Метод | Тип | Описание |
|---|---|---|
| `server.name()` | `String` | имя записи из списка серверов, `"singleplayer"` без записи |
| `server.address()` | `String` | адрес `host:port` из списка серверов, `"singleplayer"` без записи |
| `server.type()` | `ServerType` | тип подключения, MULTIPLAYER при неизвестном |
| `server.protocolVersion()` | `int` | версия протокола из записи списка серверов, 0 без записи |
| `server.pingMs()` | `int` | твоя задержка из таблиста в мс, 0 когда недоступна |
| `server.tps()` | `float` | измеренный тикрейт сервера, округление до 0.1, 20.0 вне мира |
| `server.scoreboard()` | `Scoreboard` | клиентское представление табло |
| `server.tabList()` | `TabList` | представление списка игроков |
| `server.bossBars()` | `List<BossBar>` | боссбары на HUD, пусто без игрового HUD |

### ServerType

| Константа | Описание |
|---|---|
| `SINGLEPLAYER` | встроенный сервер, записи в списке нет |
| `LAN` | сервер, найденный по LAN |
| `REALM` | Minecraft Realm |
| `MULTIPLAYER` | любой другой удалённый сервер, а также запасной вариант |

## Табло сбоку

| Метод | Тип | Описание |
|---|---|---|
| `scoreboard.visible()` | `boolean` | true, когда цель сайдбара найдена |
| `scoreboard.title()` | `String` | обычный текст заголовка сайдбара, `""` когда нет |
| `scoreboard.lines()` | `List<String>` | обычный текст нескрытых строк сайдбара, в порядке табло |
| `scoreboard.contains(vararg needles)` | `boolean` | подстрока без учёта регистра только по заголовку, не по строкам |

Цель сайдбара — слот цвета твоей команды, с откатом на обычный слот `sidebar`.

## Цели и очки

| Метод | Тип | Описание |
|---|---|---|
| `scoreboard.objectives()` | `List<Objective>` | все цели, известные клиенту, пусто вне мира |
| `scoreboard.objective(name)` | `Objective?` | цель по внутреннему имени, null когда нет |
| `scoreboard.display(slot)` | `Objective?` | цель в слоте отображения: `list`, `sidebar`, `below_name`, `sidebar.team.red` |
| `scoreboard.holders()` | `List<String>` | все известные имена владельцев очков, пусто вне мира |

### Objective

| Метод | Тип | Описание |
|---|---|---|
| `objective.name()` | `String` | внутреннее имя цели |
| `objective.displayName()` | `Text` | оформленное отображаемое имя |
| `objective.criterion()` | `String` | имя критерия, например `dummy`, `health`, `playerKillCount` |
| `objective.renderType()` | `String` | `integer` или `hearts` |
| `objective.slots()` | `List<String>` | слоты отображения, занятые целью, пусто вне мира |
| `objective.score(holder)` | `int` | очки этого владельца, 0 когда записи нет |
| `objective.entries()` | `List<ScoreEntry>` | все записи очков этой цели |

### ScoreEntry

| Метод | Тип | Описание |
|---|---|---|
| `entry.owner()` | `String` | имя владельца очков |
| `entry.value()` | `int` | числовое значение очков |
| `entry.hidden()` | `boolean` | true, когда сервер пометил запись как не отображаемую |
| `entry.name()` | `Text` | оформленное имя так, как рисуется на сайдбаре |
| `entry.formattedValue()` | `Text` | оформленное значение в собственном числовом формате записи |

`Text` описан на странице [Оформленный текст](../ui/text.md).

## Команды

| Метод | Тип | Описание |
|---|---|---|
| `scoreboard.teams()` | `List<Team>` | все команды, пусто вне мира |
| `scoreboard.team(name)` | `Team?` | команда по внутреннему имени, null когда нет |
| `scoreboard.teamOf(holder)` | `Team?` | команда, которой принадлежит это имя владельца очков |

### Team

| Метод | Тип | Описание |
|---|---|---|
| `team.name()` | `String` | внутреннее имя команды |
| `team.displayName()` | `Text` | оформленное отображаемое имя команды |
| `team.prefix()` | `Text` | оформленный префикс перед именами участников |
| `team.suffix()` | `Text` | оформленный суффикс после имён участников |
| `team.color()` | `String` | имя цвета форматирования, например `red`, `aqua` |
| `team.colorRgb()` | `int` | ARGB с альфой 0xFF, -1 когда у цвета нет RGB |
| `team.members()` | `List<String>` | имена участников, неизменяемая копия |
| `team.friendlyFire()` | `boolean` | ванильный флаг урона по своим |
| `team.showFriendlyInvisibles()` | `boolean` | ванильный флаг видимости невидимых союзников |
| `team.nametagVisibility()` | `String` | `always`, `never`, `hide_for_other_teams`, `hide_for_own_team` |
| `team.deathMessageVisibility()` | `String` | те же четыре значения, для сообщений о смерти |
| `team.collisionRule()` | `String` | `always`, `never`, `push_other_teams`, `push_own_team` |
| `team.decorate(value)` | `Text` | префикс + значение + суффикс с форматированием команды, null это `""` |
| `team.contains(holder)` | `boolean` | проверка членства, false для null |

Табло только на чтение; ничего из записанного в эти объекты до сервера не доходит.

## Боссбары

| Метод | Тип | Описание |
|---|---|---|
| `bar.uuid()` | `String` | строка UUID боссбара |
| `bar.name()` | `Text` | оформленный заголовок боссбара |
| `bar.percent()` | `float` | доля заполнения, 0..1 |
| `bar.color()` | `String` | `pink`, `blue`, `red`, `green`, `yellow`, `purple`, `white` |
| `bar.style()` | `String` | `progress`, `notched_6`, `notched_10`, `notched_12`, `notched_20` |

## Таблист

| Метод | Тип | Описание |
|---|---|---|
| `tab.header()` | `String` | обычный текст шапки таба, `""` когда нет |
| `tab.footer()` | `String` | обычный текст футера таба, `""` когда нет |
| `tab.headerContains(vararg needles)` | `boolean` | подстрока без учёта регистра по обрезанной шапке |
| `tab.footerContains(vararg needles)` | `boolean` | та же проверка по футеру |
| `tab.players()` | `List<TabEntry>` | весь список в ванильном порядке таба, пусто без подключения |

### TabEntry

| Метод | Тип | Описание |
|---|---|---|
| `row.name()` | `String` | имя профиля |
| `row.uuid()` | `String` | строка UUID профиля |
| `row.displayName()` | `String` | обычный текст имени в табе, откат на `name()` |
| `row.pingMs()` | `int` | задержка этой записи в миллисекундах |
| `row.pingMs(value)` | `void` | запись задержки только на клиенте, следующее обновление сервера затирает (API 2) |
| `row.gameMode()` | `GameMode` | SURVIVAL, CREATIVE, ADVENTURE, SPECTATOR; SURVIVAL когда не задан |
| `row.listed()` | `boolean` | true, когда таб реально рисует эту строку |
| `row.skinTexture()` | `Texture?` | текстура тела скина, null без скинов или в StreamerMode |
| `row.player()` | `PlayerEntity?` | прогруженная сущность с этим UUID, null вне мира или без прогрузки |

Строка — это запись профиля, а не сущность: в списке все игроки, которых прислал сервер, прогружены они или нет.
`Texture` описана на странице [Рендер 2D](../ui/render-2d.md), `PlayerEntity` и `GameMode` — на [Сущности и фильтры](entities.md).

## Экраны

| Метод | Тип | Описание |
|---|---|---|
| `screen.kind()` | `ScreenKind` | классификация открытого экрана |
| `screen.title()` | `Text` | оформленный заголовок экрана |
| `screen.handled()` | `boolean` | true, когда у экрана есть обработчик слотов |
| `screen.syncId()` | `int` | sync id обработчика экрана, 0 без обработчика |
| `screen.size()` | `int` | число слотов обработчика, 0 без обработчика |

### ScreenKind

| Константа | Описание |
|---|---|
| `NONE` | ни один экран не открыт |
| `INVENTORY` | ванильный экран инвентаря |
| `CREATIVE` | экран креативного инвентаря |
| `CONTAINER` | любой другой экран со слотами: сундук, житель, меню магазина |
| `CHAT` | экран ввода чата |
| `OTHER` | любой другой экран: меню, настройки |

`game.screen()` возвращает null, когда ничего не открыто; `game.screenOpen()`, `game.screenKind()` и `game.closeScreen()` — на странице [Как устроен скрипт](../start/lifecycle.md). На закрытие экрана срабатывает `ScreenCloseEvent`.

## Переводы

`game.translate(key, args)`, `game.hasTranslation(key)` и `game.language()` — на странице [Как устроен скрипт](../start/lifecycle.md).
