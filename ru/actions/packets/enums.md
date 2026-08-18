# Перечисления пакетов

34 перечисления, 180 констант — типы полей записей пакетов из раздела [Пакеты](../packets.md). Сопоставление идёт по имени константы в обе стороны: ванильная константа без пары здесь декодируется в `null`, а константа без ванильной пары срывает отправку.

```kotlin
on<PacketSendEvent> { e ->
    val packet = e.packet()
    if (packet !is C2SPlayerInteractEntityPacket) return@on
    if (packet.type() == InteractType.ATTACK) {
        log.info("атака по сущности ${packet.entityId()}")
    }
}
```

Внутри поля `List<...>` несопоставленная константа выбрасывается из списка, а не превращается в `null`.
`InteractType` — единственное исключение из сопоставления по имени: он собирается через ванильный визитор `PlayerInteractEntityC2SPacket.Handler` при декодировании и разбирается по ветвям при отправке.

## AdvancementTabAction

Поле `C2SAdvancementTabPacket`, ваниль `AdvancementTabC2SPacket.Action`. Только на приём — у этого пакета нет билдера отправки.

| Константа | Описание |
|---|---|
| `OPENED_TAB` | открыта вкладка достижений |
| `CLOSED_SCREEN` | экран достижений закрыт |

## BlockMirror

Поле `C2SUpdateStructureBlockPacket`, ваниль `net.minecraft.util.BlockMirror`.

| Константа | Описание |
|---|---|
| `NONE` | без отражения |
| `LEFT_RIGHT` | отражение по оси лево-право |
| `FRONT_BACK` | отражение по оси перёд-зад |

## BlockRotation

Поле `C2SUpdateStructureBlockPacket`, ваниль `net.minecraft.util.BlockRotation`.

| Константа | Описание |
|---|---|
| `NONE` | 0 градусов |
| `CLOCKWISE_90` | 90 градусов по часовой |
| `CLOCKWISE_180` | 180 градусов |
| `COUNTERCLOCKWISE_90` | 90 градусов против часовой, 270 по часовой |

## BossBarAction

Поле `S2CBossBarPacket`, ваниль `BossBarS2CPacket.Type`.

| Константа | Описание |
|---|---|
| `ADD` | создана новая полоса босса |
| `REMOVE` | полоса босса убрана |
| `UPDATE_PROGRESS` | изменилась заполненность, 0..1 |
| `UPDATE_NAME` | изменился текст заголовка |
| `UPDATE_STYLE` | изменились цвет полосы и деление на насечки |
| `UPDATE_PROPERTIES` | изменились флаги затемнения неба, музыки дракона и тумана |

## ChatSuggestionAction

Поле `S2CChatSuggestionsPacket`, ваниль `ChatSuggestionsS2CPacket.Action`.

| Константа | Описание |
|---|---|
| `ADD` | добавить записи к текущему списку |
| `REMOVE` | убрать записи из списка |
| `SET` | заменить весь список записями |

## ChatVisibility

Поле `C2SClientOptionsPacket`, ваниль `net.minecraft.network.message.ChatVisibility`.

| Константа | Описание |
|---|---|
| `FULL` | показывать весь чат |
| `SYSTEM` | только системные сообщения и вывод команд |
| `HIDDEN` | чат скрыт |

## ClientCommand

Поле `C2SClientCommandPacket`, ваниль `ClientCommandC2SPacket.Mode`.

| Константа | Описание |
|---|---|
| `STOP_SLEEPING` | встать с кровати |
| `START_SPRINTING` | начать спринт |
| `STOP_SPRINTING` | закончить спринт |
| `START_RIDING_JUMP` | начать зарядку прыжка лошади |
| `STOP_RIDING_JUMP` | отпустить прыжок лошади |
| `OPEN_INVENTORY` | открыть инвентарь ездового животного |
| `START_FALL_FLYING` | начать полёт на элитрах |

Билдер отправки подставляет в пакет локального игрока, поэтому без `game.player()` отправка не проходит.

## ClientStatus

Поле `C2SClientStatusPacket`, ваниль `ClientStatusC2SPacket.Mode`.

| Константа | Описание |
|---|---|
| `PERFORM_RESPAWN` | возродиться после смерти |
| `REQUEST_STATS` | запросить данные экрана статистики |

## CommandBlockType

Поле `C2SUpdateCommandBlockPacket`, ваниль `CommandBlockBlockEntity.Type`.

| Константа | Описание |
|---|---|
| `SEQUENCE` | цепочечный командный блок |
| `AUTO` | повторяющийся командный блок |
| `REDSTONE` | импульсный командный блок |

## DebugSampleType

Поле `S2CDebugSamplePacket`, ваниль `net.minecraft.util.profiler.DebugSampleType`.

| Константа | Описание |
|---|---|
| `TICK_TIME` | тайминги тика сервера, наносекунды, в списке чисел пакета |

## Difficulty

Поле `C2SUpdateDifficultyPacket` и `S2CDifficultyPacket`, ваниль `net.minecraft.world.Difficulty`.

| Константа | Описание |
|---|---|
| `PEACEFUL` | id 0, без враждебных мобов |
| `EASY` | id 1 |
| `NORMAL` | id 2 |
| `HARD` | id 3 |

## EntityAnchor

Поле `S2CLookAtPacket`, ваниль `EntityAnchorArgumentType.EntityAnchor`.

| Константа | Описание |
|---|---|
| `FEET` | точка у ног сущности, смещение по y 0 |
| `EYES` | точка на высоте глаз сущности |

## Formatting

Цвет команды в `S2CTeamPacket`, ваниль `net.minecraft.util.Formatting`.

| Константа | Описание |
|---|---|
| `BLACK` | цвет, код `0` |
| `DARK_BLUE` | цвет, код `1` |
| `DARK_GREEN` | цвет, код `2` |
| `DARK_AQUA` | цвет, код `3` |
| `DARK_RED` | цвет, код `4` |
| `DARK_PURPLE` | цвет, код `5` |
| `GOLD` | цвет, код `6` |
| `GRAY` | цвет, код `7` |
| `DARK_GRAY` | цвет, код `8` |
| `BLUE` | цвет, код `9` |
| `GREEN` | цвет, код `a` |
| `AQUA` | цвет, код `b` |
| `RED` | цвет, код `c` |
| `LIGHT_PURPLE` | цвет, код `d` |
| `YELLOW` | цвет, код `e` |
| `WHITE` | цвет, код `f` |
| `OBFUSCATED` | стиль, код `k` |
| `BOLD` | стиль, код `l` |
| `STRIKETHROUGH` | стиль, код `m` |
| `UNDERLINE` | стиль, код `n` |
| `ITALIC` | стиль, код `o` |
| `RESET` | стиль, код `r`; как цвет команды означает «без цвета» |

## InteractType

Какой из трёх вариантов несёт `C2SPlayerInteractEntityPacket`; собирается через ванильный визитор, а не по имени.

| Константа | Описание |
|---|---|
| `INTERACT` | ПКМ по сущности; `hand` задан, координаты попадания null |
| `INTERACT_AT` | ПКМ в точку сущности; `hand` и координаты попадания заданы |
| `ATTACK` | ЛКМ, атака; `hand` и координаты попадания null |

## JigsawJoint

Поле `C2SUpdateJigsawPacket`, ваниль `JigsawBlockEntity.Joint`. Только на приём — у этого пакета нет билдера отправки.

| Константа | Описание |
|---|---|
| `ROLLABLE` | присоединённая часть может вращаться вокруг оси стыка |
| `ALIGNED` | присоединённая часть держит фиксированную ориентацию |

## ParticlesMode

Поле `C2SClientOptionsPacket`, ваниль `net.minecraft.particle.ParticlesMode`.

| Константа | Описание |
|---|---|
| `ALL` | все частицы |
| `DECREASED` | уменьшенное количество частиц |
| `MINIMAL` | минимум частиц |

## PlayerAction

Поле `C2SPlayerActionPacket`, ваниль `PlayerActionC2SPacket.Action`.

| Константа | Описание |
|---|---|
| `START_DESTROY_BLOCK` | начать ломать блок в позиции пакета |
| `ABORT_DESTROY_BLOCK` | отменить начатую ломку |
| `STOP_DESTROY_BLOCK` | завершить ломку блока |
| `DROP_ALL_ITEMS` | выбросить весь held-стак |
| `DROP_ITEM` | выбросить один предмет из held-стака |
| `RELEASE_USE_ITEM` | отпустить заряжаемый или удерживаемый предмет |
| `SWAP_ITEM_WITH_OFFHAND` | поменять местами основную и вторую руку |
| `STAB` | удар булавой или копьём, добавлено в 1.21.11 |

## PlayerListAction

`S2CPlayerListPacket` несёт `List<PlayerListAction>`, ваниль `PlayerListS2CPacket.Action`.

| Константа | Описание |
|---|---|
| `ADD_PLAYER` | новая запись профиля, имя и свойства скина |
| `INITIALIZE_CHAT` | данные чат-сессии и публичного ключа |
| `UPDATE_GAME_MODE` | сменился режим игры |
| `UPDATE_LISTED` | показан или скрыт в таблисте |
| `UPDATE_LATENCY` | сменился пинг в мс |
| `UPDATE_DISPLAY_NAME` | сменилось отображаемое имя в таблисте |
| `UPDATE_LIST_ORDER` | сменился явный порядок сортировки |
| `UPDATE_HAT` | сменилась видимость слоя шляпы |

## PositionFlag

`S2CEntityPositionPacket` несёт `List<PositionFlag>`, ваниль `net.minecraft.network.packet.s2c.play.PositionFlag`.

| Константа | Описание |
|---|---|
| `X` | x относительно текущего x |
| `Y` | y относительный |
| `Z` | z относительный |
| `Y_ROT` | yaw относительный, градусы |
| `X_ROT` | pitch относительный, градусы |
| `DELTA_X` | x-скорость относительная |
| `DELTA_Y` | y-скорость относительная |
| `DELTA_Z` | z-скорость относительная |
| `ROTATE_DELTA` | текущая скорость поворачивается на дельту поворота |

## RecipeBookCategory

Поле `C2SRecipeCategoryOptionsPacket`, ваниль `net.minecraft.recipe.book.RecipeBookType`.

| Константа | Описание |
|---|---|
| `CRAFTING` | книга верстака и инвентаря |
| `FURNACE` | книга печи |
| `BLAST_FURNACE` | книга плавильной печи |
| `SMOKER` | книга коптильни |

## ResourcePackStatus

Поле `C2SResourcePackStatusPacket`, ваниль `ResourcePackStatusC2SPacket.Status`. Только на приём — у этого пакета нет билдера отправки.

| Константа | Описание |
|---|---|
| `SUCCESSFULLY_LOADED` | пак применён |
| `DECLINED` | пользователь отказался от пака |
| `FAILED_DOWNLOAD` | загрузка не удалась |
| `ACCEPTED` | пользователь согласился, загрузка начинается |
| `DOWNLOADED` | загрузка закончена, пак ещё не применён |
| `INVALID_URL` | ссылку на пак не удалось разобрать |
| `FAILED_RELOAD` | перезагрузка ресурсов после применения не удалась |
| `DISCARDED` | пак отброшен без применения |

## ScoreboardRenderType

Поле `S2CScoreboardObjectiveUpdatePacket`, ваниль `ScoreboardCriterion.RenderType`.

| Константа | Описание |
|---|---|
| `INTEGER` | счёт рисуется числом |
| `HEARTS` | счёт рисуется сердцами |

## ScoreboardSlot

Поле `S2CScoreboardDisplayPacket`, ваниль `net.minecraft.scoreboard.ScoreboardDisplaySlot`.

| Константа | Описание |
|---|---|
| `LIST` | таблист |
| `SIDEBAR` | общая боковая панель |
| `BELOW_NAME` | под ником игрока |
| `TEAM_BLACK` | панель только для чёрной команды |
| `TEAM_DARK_BLUE` | панель для тёмно-синей команды |
| `TEAM_DARK_GREEN` | панель для тёмно-зелёной команды |
| `TEAM_DARK_AQUA` | панель для тёмно-бирюзовой команды |
| `TEAM_DARK_RED` | панель для тёмно-красной команды |
| `TEAM_DARK_PURPLE` | панель для тёмно-фиолетовой команды |
| `TEAM_GOLD` | панель для золотой команды |
| `TEAM_GRAY` | панель для серой команды |
| `TEAM_DARK_GRAY` | панель для тёмно-серой команды |
| `TEAM_BLUE` | панель для синей команды |
| `TEAM_GREEN` | панель для зелёной команды |
| `TEAM_AQUA` | панель для бирюзовой команды |
| `TEAM_RED` | панель для красной команды |
| `TEAM_LIGHT_PURPLE` | панель для светло-фиолетовой команды |
| `TEAM_YELLOW` | панель для жёлтой команды |
| `TEAM_WHITE` | панель для белой команды |

## SlotActionType

Поле `C2SClickSlotPacket`, ваниль `net.minecraft.screen.slot.SlotActionType`. Только на приём — у этого пакета нет билдера отправки.

| Константа | Описание |
|---|---|
| `PICKUP` | обычный клик ЛКМ или ПКМ: взять или положить |
| `QUICK_MOVE` | перенос по шифт-клику |
| `SWAP` | обмен с хотбаром или второй рукой; `button` — индекс хотбара, 40 = вторая рука |
| `CLONE` | креативное клонирование средней кнопкой |
| `THROW` | клавиша выброса; `button` 0 = один предмет, 1 = весь стак |
| `QUICK_CRAFT` | перетаскивание с раскладкой по слотам |
| `PICKUP_ALL` | сбор одинаковых предметов двойным кликом |

## SoundCategory

Поле `S2CPlaySoundPacket`, `S2CPlaySoundFromEntityPacket` и `S2CStopSoundPacket`, ваниль `net.minecraft.sound.SoundCategory`.

| Константа | Описание |
|---|---|
| `MASTER` | общий канал |
| `MUSIC` | фоновая музыка |
| `RECORDS` | проигрыватель и нотные блоки |
| `WEATHER` | дождь и гром |
| `BLOCKS` | звуки блоков |
| `HOSTILE` | звуки враждебных мобов |
| `NEUTRAL` | звуки дружелюбных мобов |
| `PLAYERS` | звуки игроков |
| `AMBIENT` | окружение |
| `VOICE` | голос и речь |
| `UI` | звуки интерфейса |

## StructureBlockAction

Поле `C2SUpdateStructureBlockPacket`, ваниль `StructureBlockBlockEntity.Action`.

| Константа | Описание |
|---|---|
| `UPDATE_DATA` | записать настройки, ничего не делая |
| `SAVE_AREA` | сохранить выделенную область в шаблон |
| `LOAD_AREA` | поставить шаблон в мир |
| `SCAN_AREA` | определить границы структуры по угловым блокам |

## StructureBlockMode

Поле `C2SUpdateStructureBlockPacket`, ваниль `net.minecraft.block.enums.StructureBlockMode`.

| Константа | Описание |
|---|---|
| `SAVE` | режим сохранения |
| `LOAD` | режим загрузки |
| `CORNER` | режим углового маркера |
| `DATA` | режим маркера данных |

## TeamCollisionRule

Поле `S2CTeamPacket`, null когда пакет не несёт данных команды, ваниль `AbstractTeam.CollisionRule`.

| Константа | Описание |
|---|---|
| `ALWAYS` | всегда сталкивается |
| `NEVER` | никогда не сталкивается |
| `PUSH_OTHER_TEAMS` | сталкивается только с игроками других команд |
| `PUSH_OWN_TEAM` | сталкивается только со своей командой |

## TeamOperation

Что `S2CTeamPacket` делает с самой командой, ваниль `TeamS2CPacket.Operation`.

| Константа | Описание |
|---|---|
| `ADD` | команда создана или её данные обновлены |
| `REMOVE` | команда удалена |

## TeamPlayerListOperation

Что `S2CTeamPacket` делает со списком участников; та же ванильная `TeamS2CPacket.Operation` на отдельном поле.

| Константа | Описание |
|---|---|
| `ADD` | перечисленные игроки вошли в команду |
| `REMOVE` | перечисленные игроки вышли из команды |

## TeamVisibilityRule

Поле `S2CTeamPacket`, null когда пакет не несёт данных команды, ваниль `AbstractTeam.VisibilityRule`.

| Константа | Описание |
|---|---|
| `ALWAYS` | ники видны всегда |
| `NEVER` | ники не видны никогда |
| `HIDE_FOR_OTHER_TEAMS` | скрыты от игроков вне команды |
| `HIDE_FOR_OWN_TEAM` | скрыты от своей же команды |

## TestBlockMode

Поле `C2SSetTestBlockPacket`, ваниль `net.minecraft.block.enums.TestBlockMode`.

| Константа | Описание |
|---|---|
| `START` | запускает прогон теста |
| `LOG` | пишет своё сообщение при подаче сигнала |
| `FAIL` | проваливает тест при подаче сигнала |
| `ACCEPT` | отмечает тест пройденным при подаче сигнала |

## TestInstanceBlockAction

Поле `C2STestInstanceBlockActionPacket`, ваниль `TestInstanceBlockActionC2SPacket.Action`. Только на приём — у этого пакета нет билдера отправки.

| Константа | Описание |
|---|---|
| `INIT` | создать область тестовой структуры |
| `QUERY` | запросить у сервера текущее состояние блока |
| `SET` | записать изменённые настройки |
| `RESET` | сбросить тестовый экземпляр |
| `SAVE` | сохранить тестовую структуру |
| `EXPORT` | выгрузить структуру в файл |
| `RUN` | запустить тест |

## WaypointOperation

Что `S2CWaypointPacket` делает с отслеживаемой точкой на локаторной полосе, ваниль `WaypointS2CPacket.Operation`.

| Константа | Описание |
|---|---|
| `TRACK` | начать отслеживать точку |
| `UNTRACK` | перестать отслеживать точку |
| `UPDATE` | обновить уже отслеживаемую точку |
